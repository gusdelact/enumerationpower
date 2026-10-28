# Nmap: rendimiento y control de ruido

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 8.

El rendimiento importa en redes grandes o con poco ancho de banda. Puedes controlar velocidad,
frecuencia, timeouts, tasa de paquetes y reintentos. **Advertencia transversal:** acelerar casi siempre
tiene un coste: puedes **perder hosts o puertos**, y un escaneo demasiado agresivo hace que los sistemas
de seguridad te **bloqueen**.

## Timeouts (RTT)

Nmap espera un Round-Trip-Time por respuesta (por defecto arranca en ~100 ms).

```bash
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

| Opción | Uso | Riesgo |
|---|---|---|
| `--initial-rtt-timeout <t>` | RTT inicial | Muy bajo = se pasan por alto hosts. |
| `--max-rtt-timeout <t>` | RTT máximo | |

En el ejemplo del módulo el escaneo optimizado tardó **1/4** del tiempo pero encontró **2 hosts menos**.

## Reintentos

```bash
sudo nmap 10.129.2.0/24 -F --max-retries 0
```

Por defecto `--max-retries` es **10**. Bajarlo a `0` acelera pero, si un puerto no responde al primer
intento, Nmap lo salta: puedes **perder puertos** abiertos lentos.

## Tasa de paquetes (rate)

Útil en pruebas white-box donde conoces el ancho de banda o estás en allowlist:

```bash
sudo nmap 10.129.2.0/24 -F --min-rate 300
```

| Opción | Uso |
|---|---|
| `--min-rate <n>` | Mínimo de paquetes por segundo que intenta mantener. |
| `--min-parallelism <n>` | Paralelismo mínimo de sondas. |

## Plantillas de timing `-T 0..5`

Cuando no puedes ajustar a mano (p. ej. black-box), usa una plantilla. El defecto es `-T 3`.

| Plantilla | Nombre | Carácter |
|---|---|---|
| `-T0` | paranoid | Muy lento, sigiloso. |
| `-T1` | sneaky | Lento. |
| `-T2` | polite | Moderado, cuida el ancho de banda. |
| `-T3` | normal | **Por defecto.** |
| `-T4` | aggressive | Rápido. |
| `-T5` | insane | Muy rápido y **ruidoso**. |

> **En laboratorio/producción:** evita `-T4`/`-T5` y escaneos agresivos al inicio. La enumeración busca
> descubrir, no tumbar servicios ni provocar alertas. Si el entorno tiene IDS/IPS, ve a
> `nmap-evasion-firewall-ids.md`.

Valores exactos de cada plantilla: https://nmap.org/book/performance-timing-templates.html
Más rendimiento: https://nmap.org/book/man-performance.html
