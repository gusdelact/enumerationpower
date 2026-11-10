# Servicios de infraestructura (secciones 6–12)

Playbooks de enumeración por servicio: FTP, SMB, NFS, DNS, SMTP, IMAP/POP3 y SNMP. Para cada
uno: qué es, puertos, misconfiguraciones típicas, comandos y **qué documentar**. Reemplaza
`$TARGET` por la IP autorizada. Aplica OODA: observa la salida cruda, oriéntala con la config
por defecto del servicio, decide el paso menos intrusivo y actúa guardando la evidencia.

> Idea central: cada servicio existe por una razón y tiene **configuración por defecto** y
> **ajustes peligrosos** conocidos. Entiende el "porqué" desde la perspectiva del administrador.
> No explotamos: identificamos y documentamos. Verifica a mano lo que digan las herramientas.

---

## FTP — TCP 21 (control), 20 (datos) · Sección 6

**Qué es.** Transferencia de archivos en texto plano, dos canales. Modo **activo** (cliente
abre) vs **pasivo** (servidor anuncia puerto, útil tras firewall). Servidor común: **vsFTPd**
(`/etc/vsftpd.conf`); `/etc/ftpusers` deniega usuarios. **TFTP** es UDP y **sin autenticación**.

**Ajustes peligrosos:** login `anonymous` (`anonymous_enable=YES`), subida anónima
(`anon_upload_enable`, `write_enable`). Subir a un FTP ligado a un web server → posible RCE.

```bash
sudo nmap -sV -p21 -sC -A $TARGET
sudo nmap -sV -p21 -sC -A $TARGET --script-trace
find / -type f -name ftp* 2>/dev/null | grep scripts   # listar NSE de ftp

# Login anónimo (usuario: anonymous, contraseña: cualquiera)
ftp $TARGET
#   ftp> status      # resumen de configuración
#   ftp> debug ; trace
#   ftp> ls -R       # listado recursivo si ls_recurse_enable=YES
#   ftp> get "Important Notes.txt"
#   ftp> put testupload.txt   # probar subida

# Descargar TODO recursivamente
wget -m --no-passive ftp://anonymous:anonymous@$TARGET

# FTP sobre TLS
openssl s_client -connect $TARGET:21 -starttls ftp
```

**Documenta:** banner y versión, si el login anónimo está habilitado, el contenido listado y
cualquier **pista de credenciales** en los archivos.

---

## SMB — TCP 139 (NetBIOS), 445 (CIFS) · Sección 7

**Qué es.** Server Message Block: archivos, directorios e impresoras con ACLs por share.
**Samba** = implementación Unix (implementa CIFS). Config `/etc/samba/smb.conf`. **Null
session** (`-N`) = acceso anónimo sin credenciales.

**Ajustes peligrosos:** `browseable=yes`, `writable=yes`, `guest ok=yes`,
`enable privileges=yes`, `create mask=0777`, `logon/magic script`.

```bash
sudo nmap -sV -sC -p139,445 $TARGET

# Listar shares con null session
smbclient -N -L //$TARGET
smbclient -N //$TARGET/notes      # conectar; luego: ls, get <archivo>, !<cmd>

# rpcclient (MS-RPC)
rpcclient -U "" $TARGET
#   srvinfo | enumdomains | querydominfo | netshareenumall
#   netsharegetinfo <share> | enumdomusers | queryuser <RID> | querygroup <RID>

# Fuerza bruta de RIDs para enumerar usuarios
for i in $(seq 500 1100); do \
  rpcclient -N -U "" $TARGET -c "queryuser 0x$(printf '%x\n' $i)" \
  | grep "User Name\|user_rid\|group_rid" && echo ""; done

# Otras herramientas
samrdump.py $TARGET
smbmap -H $TARGET
crackmapexec smb $TARGET --shares -u '' -p ''
enum4linux-ng -A $TARGET          # si falta: git clone cddmp/enum4linux-ng
```

**Documenta:** acceso de invitado a shares, su contenido y los usuarios del workgroup.

---

## NFS — TCP 111 (rpcbind), 2049 (nfsd) · Sección 8

**Qué es.** Sistema de archivos de red (Unix). Sobre ONC-RPC (puerto 111). NFSv3 **no tiene
autenticación propia** (confía en UID/GID); NFSv4 añade auth de usuario y un solo puerto 2049.
Config: `/etc/exports`.

**Ajustes peligrosos:** `rw`, `insecure` (puertos >1024), `nohide`, **`no_root_squash`**
(archivos de root conservan UID/GID 0 → escalada subiendo binario SUID).

```bash
sudo nmap -sV -p111,2049 --script nfs* $TARGET
sudo nmap -sV -p111,2049 $TARGET

# Listar exports
showmount -e $TARGET

# Montar y revisar
mkdir -p ~/target-nfs
sudo mount -t nfs $TARGET:/ ~/target-nfs -o nolock   # o la ruta exacta del export
ls -l ~/target-nfs
ls -n ~/target-nfs            # UID/GID numéricos -> relaciona con no_root_squash
cd ~ && sudo umount ~/target-nfs
```

**Documenta:** el export abierto, su contenido y los permisos (UID/GID) observados.

---

## DNS — UDP/TCP 53 · Sección 9

**Qué es.** Resolución de nombres. Tipos de servidor: Root, Authoritative, Caching, Forwarding,
Resolver. Registros: A, AAAA, MX, NS, TXT, CNAME, PTR, SOA. Config BIND9 (`named.conf`, zone
files). **Ajuste peligroso estrella:** `allow-transfer { any; }` → **AXFR** descarga **toda la
zona**, revelando hosts/IPs internos. Otros: `allow-query`, `allow-recursion`.

```bash
# Name servers de la zona
dig ns inlanefreight.htb @$TARGET

# Versión de BIND
dig CH TXT version.bind @$TARGET

# Todos los registros / SOA
dig any inlanefreight.htb @$TARGET
dig soa www.inlanefreight.com

# Zone transfer (AXFR) — prueba la zona pública y las internas
dig axfr inlanefreight.htb @$TARGET
dig axfr internal.inlanefreight.htb @$TARGET

# Brute-force de subdominios
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do \
  dig $sub.inlanefreight.htb @$TARGET | grep -v ';\|SOA' | sed -r '/^\s*$/d' \
  | grep $sub | tee -a subdomains.txt; done

# DNSenum
dnsenum --dnsserver $TARGET --enum -p 0 -s 0 -o subdomains.txt \
  -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

**Documenta:** si el AXFR tiene éxito, los **hosts internos** descubiertos y los TXT. Si está
cerrado, documéntalo como configuración correcta.

---

## SMTP — TCP 25 (465 SSL, 587 submission) · Sección 10

**Qué es.** Envío de correo: MUA → MSA → MTA → (relay) → MDA → buzón. ESMTP = SMTP + STARTTLS +
AUTH. Protecciones: SPF, DKIM, DMARC. **Ajuste peligroso:** **open relay**
(`mynetworks = 0.0.0.0/0`) → spoofing/spam. `VRFY`/`EXPN` permiten **enumerar usuarios** (ojo:
código 252 da falsos positivos).

```bash
sudo nmap -sC -sV -p25 $TARGET
sudo nmap -p25 --script smtp-open-relay -v $TARGET

# Interacción manual
telnet $TARGET 25
#   EHLO inlanefreight.htb
#   VRFY robin           # compara respuesta de usuario real vs inexistente
#   MAIL FROM: <...> ; RCPT TO: <...> ; DATA ; QUIT

# A través de proxy web:  CONNECT <ip>:25 HTTP/1.0
```

**Documenta:** si `VRFY` está habilitado, la lista de usuarios válidos y si es open relay.

---

## IMAP / POP3 — 143/993 · 110/995 · Sección 11

**Qué es.** Lectura de correo. IMAP sincroniza carpetas multi-cliente; POP3 solo recupera/borra.
Servidor común: **Dovecot**. Puertos altos (993/995) usan TLS. El certificado revela
organización, FQDN y email del admin. **Ajustes peligrosos:** `auth_debug`,
`auth_debug_passwords`, `auth_verbose(_passwords)`, `auth_anonymous_username`.

```bash
sudo nmap -sV -sC -p110,143,993,995 $TARGET

# Con credenciales débiles descubiertas antes (p. ej. robin:robin)
curl -k "imaps://$TARGET" --user robin:robin
curl -k "imaps://$TARGET" --user robin:robin -v     # ver TLS, cert, banner

# Interacción TLS cruda
openssl s_client -connect $TARGET:imaps
#   a LOGIN robin robin
#   a LIST "" *
#   a SELECT INBOX
#   a FETCH 1 all
#   a LOGOUT

openssl s_client -connect $TARGET:pop3s
#   USER robin / PASS robin / STAT / LIST / RETR 1 / QUIT
```

**Documenta:** datos del certificado, login con credenciales reutilizadas y contenido del buzón.

---

## SNMP — UDP 161 (traps 162) · Sección 12

**Qué es.** Monitoreo/gestión de dispositivos de red. **MIB** (base de info en ASN.1) → **OID**
(identificadores jerárquicos). **v1/v2c**: sin cifrado, **community string** en texto plano
(como una contraseña). **v3**: auth usuario/contraseña + cifrado. Config `/etc/snmp/snmpd.conf`.

**Ajustes peligrosos:** `rwuser noauth`, `rwcommunity <string> <IPv4>`, `rwcommunity6`.

```bash
# Consultar OIDs con una community string
snmpwalk -v2c -c public $TARGET

# Fuerza bruta de community strings (SecLists)
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt $TARGET

# Fuerza bruta de OIDs (requiere community conocida)
sudo apt install braa
braa <community>@$TARGET:.1.3.6.*
```

**Documenta:** versión SNMP, community strings válidas y la información del dispositivo expuesta
(usuarios, procesos, software, config de red).

> **Nota del lab local:** el laboratorio de este proyecto **no** incluye SNMP; esta sección es
> parte del curso completo y aplica a otros objetivos autorizados.

---

## Resumen de puertos (secciones 6–12)

| Puerto | Servicio | Sección |
|---|---|---|
| 21 / 20 (+pasivo) | FTP | 6 |
| 139 / 445 | SMB | 7 |
| 111 / 2049 (+20048 mountd) | NFS | 8 |
| 53 | DNS | 9 |
| 25 / 465 / 587 | SMTP | 10 |
| 110 / 143 / 993 / 995 | POP3 / IMAP | 11 |
| 161 / 162 (UDP) | SNMP | 12 |
