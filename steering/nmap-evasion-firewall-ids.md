# Nmap: firewall e IDS/IPS (reconocimiento y evasión)

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 9.
> Técnicas para redes **autorizadas**. El fin es entender la postura defensiva y poder enumerar sin
> ser bloqueado, no evadir controles de sistemas de terceros.

## Conceptos

- **Firewall:** decide pasar / dropear / bloquear paquetes según reglas. `filtered` suele ser su huella.
- **IDS:** detecta y **reporta** patrones de ataque (pattern matching / firmas).
- **IPS:** complementa al IDS tomando **acciones defensivas** automáticas (p. ej. bloquear tu IP).

## Reconocer reglas de firewall: drop vs reject

Los paquetes rechazados dan respuesta explícita; los dropeados, no:

- **Drop:** silencio (sin respuesta).
- **Reject:** TCP con flag **RST**, o ICMP con códigos de error:
  Net/Host/Port/Proto Unreachable, Net/Host Prohibited.

## ACK scan (`-sA`): mapear el firewall

El ACK scan es **más difícil de filtrar** que SYN/Connect: envía solo el flag ACK. El firewall a menudo
deja pasar el ACK porque no sabe si la conexión nació dentro o fuera. Un puerto abierto o cerrado debe
responder con RST.

```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

Comparando los `RCVD`:

| Escaneo | Puerto abierto | Puerto filtrado por drop |
|---|---|---|
| `-sS` (SYN) | responde `SYN-ACK` (SA) → `open` | nada → `filtered` |
| `-sA` (ACK) | responde `RST` (R) → `unfiltered` | nada → `filtered` |

Si con `-sA` un puerto sale `unfiltered`, el ACK atravesó el firewall: pista de qué deja pasar la regla.

## Detectar IDS/IPS

Los IDS/IPS son pasivos, difíciles de detectar directamente. Método práctico: **escanea desde un único
VPS**; si en algún momento ese host pierde el acceso al objetivo, hay un IPS que te bloqueó por la IP de
origen. Entonces baja el ruido y continúa desde **otro VPS**. Un escaneo agresivo de un solo puerto puede
provocar a propósito la reacción del sistema para confirmar que hay monitorización.

> Si el panel del lab muestra un contador de alertas, vigílalo tras cada comando y para si sube.

## Decoys (`-D`): disfrazar el origen

Inserta IPs señuelo aleatorias en la cabecera IP; tu IP real queda mezclada entre ellas. Las señuelo
**deben estar vivas** o los mecanismos anti SYN-flood pueden dejar el servicio inaccesible.

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

Sirve para SYN, ACK, ICMP y detección de SO. Ojo: los ISPs/routers suelen filtrar paquetes spoofeados.

## Source IP (`-S`) y source port (`--source-port`)

Si solo ciertas subredes tienen acceso, prueba a fijar la IP de origen y la interfaz; o aprovecha un
puerto de origen "de confianza" (DNS 53) que el firewall deje pasar.

```bash
# Mismo host, distinto source IP: 445 pasa de filtered a open
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

# ¿El firewall confía en el puerto origen 53?
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
ncat -nv --source-port 53 10.129.2.28 50000
```

| Opción | Uso |
|---|---|
| `-sA` | ACK scan: descubre reglas de firewall. |
| `-D RND:5` | 5 IPs señuelo aleatorias (deben estar vivas). |
| `-S <IP> -e <iface>` | IP de origen distinta por una interfaz concreta. |
| `--source-port 53` | Puerto de origen "confiable" (DNS). |
| `--dns-server <ns>` | Usar DNS propios (útil desde una DMZ). |

## Enfoque de los labs de evasión

- Un puerto, un objetivo y un comando a la vez. Nada de `-A`, `-T4/-T5` ni `--script vuln` al inicio.
- Usa siempre `-Pn -n --disable-arp-ping` para no generar tráfico extra.
- Para versión o SO, el banner suele bastar: prefiere `nc`/`ncat` antes que `-sV` masivo.
- Si te bloquean, reinicia el target antes de seguir.
- Para un servicio detrás de un puerto `filtered`: encuéntralo con `-p-` (con calma, `--min-rate`
  moderado), prueba si el firewall confía en el source port 53 y lee el banner a mano con `ncat`.
