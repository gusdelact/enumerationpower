# Nmap: metodología y flujo de escaneo

> Guía de steering del power **footprinting-enumeration**, derivada del módulo *Network
> Enumeration with Nmap* (HTB Academy, secciones 1–9). Complementa al skill `footprinting`
> (`references/nmap.md`): el skill resume el cheat sheet; estas guías de steering desarrollan
> el **flujo de trabajo por fase** y el razonamiento detrás de cada opción.
>
> **Encuadre:** solo se escanean activos dentro del alcance autorizado (laboratorio propio o
> permiso por escrito). El objetivo de la enumeración es **descubrir y documentar**, no explotar.

## Por qué enumerar bien antes de actuar

La enumeración es la parte más crítica: el objetivo **no** es entrar al sistema, sino identificar
**todas** las vías por las que se podría atacar. Las herramientas solo valen si sabes qué hacer con
la información que devuelven; no sustituyen el conocimiento del protocolo ni la atención al detalle.

Dos cosas buscamos siempre en cada servicio:

1. **Funciones/recursos** que permitan interactuar con el objetivo o den información adicional.
2. **Información** que a su vez lleve a más información para acceder al objetivo.

La mayoría de los hallazgos vienen de **misconfiguraciones** o descuidos de seguridad. Confiar solo
en firewall, GPOs y parches no basta.

> **Límite de las herramientas automáticas:** un escáner usa timeouts; si un puerto no responde a
> tiempo lo marca `closed`, `filtered` o `unknown`. Un puerto marcado `closed` que Nmap oculta puede
> ser justo la vía de acceso. Por eso la **enumeración manual** (banner grabbing con `nc`, captura con
> `tcpdump`) es un componente crítico que complementa a Nmap.

## Qué es Nmap

Network Mapper: analiza redes con paquetes crudos para identificar hosts vivos, servicios y
versiones, SO, y si hay filtros/firewall/IDS. Sintaxis:

```bash
nmap <tipos de escaneo> <opciones> <objetivo>
```

Técnicas internas de Nmap: host discovery, port scanning, service/version detection, OS detection
y NSE (scripting).

## Flujo recomendado por fases (OODA en cada paso)

Cada comando es un **Act**: antes decide el menos intrusivo que resuelva tu duda; después vuelve a
**Observe** y lee la salida cruda separando hechos de inferencias.

```bash
export TARGET=<ip-autorizada>
mkdir -p evidencia                 # las salidas originales se guardan aquí, en el workspace

# Fase 1 — ¿qué hosts están vivos? (ver nmap-descubrimiento-hosts.md)
sudo nmap $TARGET/24 -sn -oA evidencia/hosts | grep for | cut -d" " -f5

# Fase 2 — escaneo rápido para una primera foto (menos ruido)
sudo nmap $TARGET --top-ports=100 -oA evidencia/quick

# Fase 3 — cobertura completa en segundo plano (ver nmap-escaneo-puertos.md)
sudo nmap $TARGET -p- -oA evidencia/full --stats-every=30s

# Fase 4 — versión + scripts por defecto SOLO sobre los puertos abiertos encontrados
sudo nmap $TARGET -p 22,25,80 -sV -sC -oA evidencia/services

# Fase 5 — verificación manual del banner (ver nmap-deteccion-servicios.md)
nc -nv $TARGET 25
```

Haz un escaneo rápido primero (menos tráfico, menos riesgo de bloqueo) y lanza el `-p-` completo en
paralelo. Guarda **siempre** con `-oA` para comparar escaneos y para el reporte.

## Mapa de guías de steering de este power

| Guía | Sección HTB | Cuándo usarla |
|---|---|---|
| `nmap-metodologia-flujo.md` | 1, 2 | Punto de partida: principios y flujo por fases. |
| `nmap-descubrimiento-hosts.md` | 3 | Saber qué hosts de un rango están vivos. |
| `nmap-escaneo-puertos.md` | 4 | Estados de puerto, SYN/Connect, UDP, drop vs reject. |
| `nmap-salida-reportes.md` | 5 | Guardar resultados y generar reporte HTML. |
| `nmap-deteccion-servicios.md` | 6 | Versión de servicio y banner grabbing manual. |
| `nmap-nse.md` | 7 | Categorías NSE y uso de scripts. |
| `nmap-rendimiento.md` | 8 | Ajustar velocidad/ruido en redes grandes o lentas. |
| `nmap-evasion-firewall-ids.md` | 9 | Firewall, IDS/IPS, ACK scan, decoys, source IP/port. |
