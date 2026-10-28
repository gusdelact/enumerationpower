# Nmap: descubrimiento de hosts (host discovery)

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 3.
> Solo sobre rangos/activos autorizados.

Antes de escanear puertos conviene saber **qué sistemas están vivos** en el rango. El método más
efectivo son los **ICMP echo requests**, pero Nmap ajusta su comportamiento según el contexto de red.

## Barrer un rango y quedarte solo con las IPs vivas

```bash
# -sn desactiva el escaneo de puertos (solo descubrimiento)
sudo nmap 10.129.2.0/24 -sn -oA evidencia/hosts | grep for | cut -d" " -f5
```

| Opción | Uso |
|---|---|
| `10.129.2.0/24` | Rango CIDR |
| `10.129.2.18-20` | Rango dentro de un octeto |
| `IP1 IP2 IP3` | Varias IPs sueltas |
| `-iL hosts.lst` | Leer los objetivos de un archivo |
| `-sn` | Desactiva el escaneo de puertos |
| `-oA <base>` | Guarda en los tres formatos |

> **Lectura (Observe):** si de 7 hosts solo 3 aparecen vivos, puede que los otros 4 **ignoren** los
> ICMP echo por su firewall. "No vivo" según Nmap no es un hecho absoluto: es "no respondió al método
> usado". Para esos casos, mira la guía `nmap-evasion-firewall-ids.md`.

## ARP vs ICMP: entender qué envía Nmap realmente

En una LAN, antes del ICMP echo Nmap suele mandar un **ARP ping** y decide por la ARP reply. Para ver
qué ocurre en la red usa `--packet-trace` y `--reason`:

```bash
# Forzar ICMP echo y ver cada paquete
sudo nmap 10.129.2.18 -sn -PE --packet-trace

# Ver por qué Nmap marca el host como vivo
sudo nmap 10.129.2.18 -sn -PE --reason

# Desactivar ARP para observar el ICMP echo puro
sudo nmap 10.129.2.18 -sn -PE --packet-trace --disable-arp-ping
```

| Opción | Uso |
|---|---|
| `-PE` | Ping con ICMP Echo Request |
| `-Pn` | No hacer ping; asume el host vivo (útil si bloquea ICMP) |
| `--disable-arp-ping` | Desactiva el ARP ping (en LAN Nmap lo usa por defecto) |
| `-n` | Sin resolución DNS (menos tráfico/ruido) |
| `--packet-trace` | Muestra cada paquete enviado/recibido |
| `--reason` | Muestra el motivo del estado asignado |

## Pista de SO por el TTL

El TTL de la respuesta ICMP es una **inferencia** barata del SO (no una prueba):

- TTL ≈ 64 → Linux/Unix
- TTL ≈ 128 → Windows

Regístralo como hipótesis y confírmalo luego con banners o `-O`/`-sV`.

Más estrategias: https://nmap.org/book/host-discovery-strategies.html
