# Nmap: detección de servicios y banner grabbing

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 6.

Determinar la **aplicación y su versión** con la mayor precisión posible es clave: con un número de
versión exacto puedes buscar vulnerabilidades y exploits que encajen con ese servicio y SO concretos.

## Flujo: rápido primero, versión después

Un `-p-` completo tarda. Haz un escaneo rápido para la primera foto (menos tráfico, menos riesgo de
bloqueo), lanza el `-p-` en segundo plano, y aplica `-sV` sobre los puertos que ya sabes abiertos.

```bash
sudo nmap 10.129.2.28 -p- -sV
```

Durante un escaneo largo, consulta el progreso:

| Acción | Efecto |
|---|---|
| `[Barra espaciadora]` | Muestra el estado puntual del escaneo. |
| `--stats-every=5s` | Imprime el progreso cada N segundos (`s`) o minutos (`m`). |
| `-v` / `-vv` | Muestra los puertos abiertos en cuanto Nmap los detecta. |

## Cómo deduce Nmap la versión

Nmap mira primero el **banner** del puerto. Si el banner no basta, usa un sistema de **firmas** (más
lento). La salida `-sV` da `PORT STATE SERVICE VERSION` y a veces una línea `Service Info:` con el host
y el SO (p. ej. `Host: inlane; OS: Linux`).

> **`-sV` puede omitir información del banner.** El escaneo automático a veces no sabe manejar lo que el
> servicio devuelve. Lo que Nmap resume **no** es necesariamente todo lo que el servicio dijo.

## Verificación manual del banner (crítica)

Tras el three-way handshake, muchos servicios mandan un banner de identificación en un paquete con flag
**PSH-ACK**. Conéctate a mano y, si quieres, intercepta el tráfico para ver exactamente qué llega:

```bash
# Terminal 1: capturar el tráfico entre tu IP y el objetivo
sudo tcpdump -i eth0 host <MI_IP> and 10.129.2.28

# Terminal 2: abrir el servicio y leer el banner crudo
nc -nv 10.129.2.28 25
# 220 inlane ESMTP Postfix (Ubuntu)   <- aquí aparece "Ubuntu", que -sV podía no mostrar
```

Flujo esperado en la captura: `SYN → SYN-ACK → ACK → PSH-ACK (banner) → ACK`. El PSH indica que el
servidor te está enviando datos; el ACK confirma la recepción.

> **Regla cognitiva:** el banner es una **inferencia** sobre el software/versión, no una vulnerabilidad
> confirmada. Además puede estar manipulado o eliminado. Regístralo con su procedencia (comando, fecha)
> y como hipótesis hasta validarlo.

Cuando tengas el servicio identificado, abre el playbook del servicio en el skill `footprinting`
(`references/servicios.md`, `bases-de-datos.md`, `gestion-remota.md`) para enumerarlo a fondo.
