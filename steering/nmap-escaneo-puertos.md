# Nmap: escaneo de puertos y estados

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 4.
> Solo sobre activos autorizados.

Una vez que el host está vivo, buscamos: puertos abiertos y sus servicios, versiones, información que
dan los servicios, y el SO. Entender **cómo** obtiene Nmap cada estado es lo que te deja leer la salida.

## Los 6 estados de un puerto

| Estado | Significado |
|---|---|
| `open` | Se estableció conexión (TCP, datagrama UDP o asociación SCTP). Es un **hecho**. |
| `closed` | TCP respondió con flag **RST**. También sirve para confirmar que el host está vivo. |
| `filtered` | Nmap no puede decidir: sin respuesta o con error ICMP. Suele indicar firewall/packet filter. |
| `unfiltered` | Solo en TCP-ACK (`-sA`): el puerto es accesible pero no se sabe si abierto o cerrado. |
| `open\|filtered` | Sin respuesta (típico en UDP). |
| `closed\|filtered` | Solo en idle scan (`-sI`): no se pudo determinar si cerrado o filtrado. |

## Selección de puertos

| Opción | Uso |
|---|---|
| (nada) | Top 1000 TCP |
| `-p 22,25,80` | Lista concreta |
| `-p 22-445` | Rango |
| `-p-` | Los 65535 |
| `--top-ports=10` | Los N más frecuentes de la base de datos de Nmap |
| `-F` | Fast scan: top 100 |

Por defecto Nmap hace SYN scan (`-sS`) sobre los top 1000 TCP **si eres root** (necesita raw sockets);
sin privilegios cae al Connect scan (`-sT`).

## SYN scan (`-sS`) vs Connect scan (`-sT`)

Para ver con claridad los paquetes, aísla el ruido con `-Pn -n --disable-arp-ping --packet-trace`.

- **SYN (`-sS`, half-open):** manda SYN y no completa el handshake. `SYN-ACK` = open, `RST` (RA) =
  closed, nada = filtered. Más sigiloso: minimiza logs de conexión (aunque IDS/IPS modernos lo detectan).
- **Connect (`-sT`, full):** completa el three-way handshake. Muy **preciso**, pero el **menos sigiloso**:
  genera logs y lo detecta fácil el IDS/IPS. A cambio interactúa "limpio" con el servicio (menos riesgo de
  romperlo), por eso a veces se le llama escaneo "polite".

```bash
sudo nmap 10.129.2.28 -p 21 -sS --packet-trace -Pn -n --disable-arp-ping
# SENT ... :21 S     (enviamos SYN)
# RCVD ... :21 > RA  (RST+ACK => closed)
```

## Puertos filtered: drop vs reject

Distinguirlos indica cómo está configurado el firewall:

| Comportamiento | Qué ves con `--packet-trace` | Pista |
|---|---|---|
| **Drop** | Varios `SENT` sin `RCVD`; el escaneo tarda (reintentos, `--max-retries` por defecto 10). | El firewall ignora el paquete. |
| **Reject** | `RCVD ICMP ... Port unreachable (type=3/code=3)`. | El firewall responde con error explícito. |

```bash
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn   # drop: ~2s, sin RCVD
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn   # reject: ICMP 3/3
```

Si sabes que el host está vivo y un puerto sale `filtered`, anótalo como candidato a revisar luego
(puede estar simplemente protegido por una regla).

## Escaneo UDP (`-sU`)

UDP es sin estado: Nmap manda datagramas vacíos y a menudo no hay respuesta, así que es **mucho más
lento** y propenso a `open|filtered`. Los administradores a veces olvidan filtrar UDP.

| Respuesta del objetivo | Estado resultante |
|---|---|
| ICMP tipo 3 código 3 (port unreachable) | `closed` |
| Respuesta de la aplicación | `open` |
| Sin respuesta / otros ICMP | `open\|filtered` |

```bash
sudo nmap 10.129.2.28 -F -sU                      # top 100 UDP (lento)
sudo nmap 10.129.2.28 -sU -p137 --packet-trace --reason -Pn -n --disable-arp-ping
```

Limita el UDP a los puertos que te interesan (53, 111, 161…) para no esperar de más.

## Plantilla de depuración de un puerto

Cuando un resultado no cuadre, aísla un único puerto y mira los paquetes:

```bash
sudo nmap <IP> -p <PUERTO> -Pn -n --disable-arp-ping --packet-trace --reason
```

Más técnicas: https://nmap.org/book/man-port-scanning-techniques.html
