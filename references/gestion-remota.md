# Gestión remota e IPMI (secciones 16–18)

Enumeración de IPMI y de los protocolos de administración remota de Linux y Windows. Reemplaza
`$TARGET` por la IP autorizada. Estos servicios dan acceso potente: enumera y documenta, no
hagas fuerza bruta prematura.

---

## IPMI — UDP 623 · Sección 16

**Qué es.** Gestión de hardware independiente del SO/BIOS. El subsistema es el **BMC**
(Baseboard Management Controller); acceder a él equivale casi a acceso físico. Comunes: HP iLO,
Dell iDRAC, Supermicro IPMI.

**Defaults:** Dell iDRAC `root:calvin` · Supermicro `ADMIN:ADMIN` · HP iLO (8 caracteres
aleatorios: mayúsculas + dígitos).

**Ajuste peligroso:** fallo en el protocolo **RAKP** de IPMI 2.0 → el servidor envía un hash
salteado de la contraseña **antes** de autenticar → crack offline con **Hashcat `-m 7300`**.

```bash
sudo nmap -sU --script ipmi-version -p623 $TARGET

# Metasploit
#   use auxiliary/scanner/ipmi/ipmi_version
#   use auxiliary/scanner/ipmi/ipmi_dumphashes      # dump de hashes (RAKP)

# Crack offline
hashcat -m 7300 ipmi.txt
# HP iLO default (8 chars mayúsc+dígitos):
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

**Documenta:** versión de IPMI, fabricante del BMC, si expone hashes RAKP y defaults aplicables.

---

## Linux Remote Management · Sección 17

### SSH — TCP 22
OpenSSH. SSH-2 > SSH-1 (SSH-1 es vulnerable a MITM). Auth por password o **clave pública**.
Config `/etc/ssh/sshd_config`. **Ajustes peligrosos:** `PasswordAuthentication yes`,
`PermitEmptyPasswords yes`, `PermitRootLogin yes`, `Protocol 1`, `X11Forwarding yes`.

```bash
# Fingerprint de algoritmos y configuración
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py $TARGET

# Ver métodos de autenticación admitidos (el banner revela versión)
ssh -v user@$TARGET
ssh -v user@$TARGET -o PreferredAuthentications=password
```

### Rsync — TCP 873
Copia eficiente (delta-transfer). Puede exponer **shares sin autenticación**; revisa reutilización
de credenciales.

```bash
sudo nmap -sV -p873 $TARGET
nc -nv $TARGET 873               # luego: @RSYNCD y #list
rsync -av --list-only rsync://$TARGET/dev
rsync -av rsync://$TARGET/dev ./         # descargar
rsync -av -e ssh rsync://$TARGET/dev ./  # sobre SSH
```

### R-services — TCP 512 (rexec), 513 (rlogin), 514 (rsh/rcp)
Obsoletos, texto plano, vulnerables a MITM. Confían en `/etc/hosts.equiv` y `.rhosts`
(el `+` es wildcard de confianza total).

```bash
sudo nmap -sV -p512,513,514 $TARGET
rlogin $TARGET -l htb-student
rwho
rusers -al $TARGET
```

**Documenta:** versiones, métodos de auth permitidos, shares rsync accesibles y archivos de
confianza (`.rhosts`) peligrosos.

---

## Windows Remote Management · Sección 18

### RDP — TCP/UDP 3389
GUI remota. TLS/SSL desde Vista, certificados self-signed por defecto; NLA (Network Level Auth).

```bash
nmap -sV -sC -p3389 --script rdp* $TARGET
nmap -sV -sC -p3389 --packet-trace --disable-arp-ping -n $TARGET

# rdp-sec-check (Perl)
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl $TARGET

# Conectar
xfreerdp /u:cry0l1t3 /p:'P455w0rd!' /v:$TARGET
```
> Ojo: la cookie `mstshash=nmap` es detectable por EDR/threat hunters.

### WinRM — TCP 5985 (HTTP) / 5986 (HTTPS)
SOAP (WS-Management). Habilitado por defecto desde Server 2012. WinRS ejecuta comandos remotos.

```bash
nmap -sV -sC -p5985,5986 --disable-arp-ping -n $TARGET
evil-winrm -i $TARGET -u <user> -p '<pass>'
# PowerShell:  Test-WsMan <host>
```

### WMI — TCP 135 (luego puerto dinámico)
Interfaz crítica de administración de Windows.

```bash
/usr/share/doc/python3-impacket/examples/wmiexec.py <user>:'<pass>'@$TARGET "hostname"
```

**Documenta:** servicios de administración expuestos, nivel de seguridad de RDP (NLA, cifrado,
certificado), y si WinRM/WMI son accesibles con las credenciales descubiertas.

---

## Resumen (secciones 16–18)

| Puerto | Servicio | Defaults / notas |
|---|---|---|
| 623 (UDP) | IPMI | iDRAC `root:calvin`, Supermicro `ADMIN:ADMIN`, iLO aleatorio; RAKP → Hashcat 7300 |
| 22 | SSH | clave pública > password; revisa `sshd_config` |
| 873 | Rsync | shares sin auth |
| 512/513/514 | R-services | obsoletos, `.rhosts` con `+` |
| 3389 | RDP | self-signed, NLA |
| 5985/5986 | WinRM | evil-winrm |
| 135 | WMI | wmiexec.py |
