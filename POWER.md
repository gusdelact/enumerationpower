---
name: "footprinting-enumeration"
displayName: "Footprinting and Enumeration"
description: "Curso completo de footprinting/enumeración (HTB Academy): principios, metodología de 6 capas, OSINT pasivo y enumeración de servicios de infraestructura y bases de datos con nmap y OODA, en laboratorios autorizados. Incluye guías de steering de nmap por fase (descubrimiento de hosts, escaneo de puertos, detección de servicios, NSE, rendimiento, evasión de firewall/IDS, salida y reportes)."
keywords: ["footprinting", "enumeration", "nmap", "reconnaissance", "osint", "pentest", "ethical-hacking"]
author: "Gustavo De la Cruz Tovar"
x-ethack:
  kind: "offensive"
  ecCouncilPhase: "Scanning"
  forensicsStage: null
  team: "red-team"
  owner: "Gustavo De la Cruz Tovar"
  tools: ["nmap"]
  allowlistValidation:
    required: true
    source: "engagements/${ENGAGEMENT_ID}/scope.yaml#assets"
    targetParams: ["target", "host", "hostname", "domain", "url"]
  executionMode:
    param: "Modo_Ejecucion"
    allowed: ["script-kiddie", "green-hat", "pro"]
    default: "script-kiddie"
  documentation:
    root: "powers/footprinting-enumeration/docs"
    required: ["prompt.md", "sessions/", "feedback.md"]
---

# Footprinting and Enumeration

## Al activar este power (obligatorio, en este orden)

1. **Modo de ejecución (perfil).** Lee el parámetro `Modo_Ejecucion` del frontmatter
   (`x-ethack.executionMode`, valores `script-kiddie` | `green-hat` | `pro`, default
   `script-kiddie`). Si el prompt no lo indica, el agente **pregunta** cuál perfil usar antes de
   escanear; si aun así no se define, usa el default `script-kiddie`:
   > ¿Con qué perfil trabajamos?
   > - **script-kiddie:** te doy el `nmap` listo y los hallazgos, con explicación mínima.
   > - **green-hat:** antes de ejecutar te explico cada flag y te pido razonar la salida.
   > - **pro:** no te doy el comando hecho; te guío a diseñar tu barrido/escáner y te pido la
   >   justificación ética y legal.
2. **Cargar `nmap-modos-ejecucion.md`** (detalle de los tres perfiles y reglas de cambio de modo).
3. **Cargar `nmap-metodologia-flujo.md`** y seguir el flujo de enumeración según el perfil elegido.

## Overview

Este power de conocimiento empaqueta el módulo de **footprinting/enumeración** de
**Hack The Box Academy** y lo convierte en un flujo que el agente puede seguir en un laboratorio
autorizado:

- **Principios** de footprinting y la **metodología de 6 capas**.
- **OSINT pasivo** como punto de partida.
- **Enumeración de servicios** de infraestructura y bases de datos con **nmap**.
- Metodología cognitiva **OODA** (Observe → Orient → Decide → Act) aplicada al reconocimiento.

> **Encuadre:** material didáctico para laboratorios autorizados o trabajos con autorización formal.
> El power no concede autorización por sí mismo. Creado para el curso de Ingeniería Cibernética
> (ULSA) con asistencia de Kiro.

## Cómo está organizado el conocimiento (steering/)

Las guías se cargan **bajo demanda** (acción `readSteering`): lee solo la que necesites según la fase.

| Guía | Contenido |
|---|---|
| `nmap-modos-ejecucion.md` | **Léela al inicio.** Los perfiles script-kiddie, green-hat y pro del parámetro `Modo_Ejecucion`, qué cambia el agente en cada uno y las reglas de cambio. |
| `nmap-metodologia-flujo.md` | **Léela primero.** Metodología y flujo de enumeración con nmap y OODA. |
| `nmap-descubrimiento-hosts.md` | Descubrimiento de hosts. |
| `nmap-escaneo-puertos.md` | Escaneo de puertos. |
| `nmap-deteccion-servicios.md` | Detección de servicios y versiones. |
| `nmap-nse.md` | Nmap Scripting Engine (NSE). |
| `nmap-rendimiento.md` | Ajuste de rendimiento y temporización. |
| `nmap-evasion-firewall-ids.md` | Evasión de firewall e IDS. |
| `nmap-salida-reportes.md` | Formatos de salida y reportes. |

## Modos de ejecución (perfil del hacker)

Además del flujo técnico, este power se ejecuta en uno de **tres perfiles** que definen cuánto hace
el agente por ti y cuánto te hace pensar. El perfil se fija con el parámetro **`Modo_Ejecucion`** del
frontmatter (`x-ethack.executionMode`: `allowed: ["script-kiddie", "green-hat", "pro"]`,
`default: "script-kiddie"`). No cambian la metodología de 6 capas, OODA, las herramientas ni la
frontera de autorización; cambia la profundidad de la explicación y quién razona. Detalle y reglas de
cambio en `steering/nmap-modos-ejecucion.md`.

| `Modo_Ejecucion` | Perfil | Qué hace el agente en footprinting-enumeration |
|---|---|---|
| `script-kiddie` *(default)* | Nivel de entrada | Da el comando listo (p. ej. `nmap -sC -sV -p- -oA enum/host 10.129.x.x`) y resume puertos, servicios y versiones, con explicación mínima. Para terminar rápido. |
| `green-hat` | Aprendiz | Antes de ejecutar, explica cada flag (`-sC`, `-sV`, `-p-`, `-oA`) y por qué ese orden, disecciona la invocación y te pide predecir e interpretar la salida (OODA). Para aprender en el laboratorio. |
| `pro` | Profesional/experto (White/Black Hat) | No da el comando hecho: te guía a diseñar tu propio barrido o escáner (Bash/Python, NSE, sockets), discute arquitectura y límites (`-sS` vs `-sT`, temporización, ruido) y exige una justificación ética y legal escrita. |

**Prompts de arranque:**

- **script-kiddie:** *Activa footprinting-enumeration con `Modo_Ejecucion=script-kiddie`. Dame el nmap para enumerar 10.129.x.x y resúmeme los servicios.*
- **green-hat:** *Activa footprinting-enumeration con `Modo_Ejecucion=green-hat`. Explícame cada flag antes de ejecutar y pregúntame qué puertos espero.*
- **pro:** *Activa footprinting-enumeration con `Modo_Ejecucion=pro`. Guíame para diseñar yo el barrido con evasión y pídeme la justificación legal.*

**Elección y cambio de modo:** si el humano no indica el perfil, el agente lo pregunta antes de
escanear (default `script-kiddie`). Subir la exigencia (script-kiddie → green-hat → pro) lo pide
cualquiera; bajarla (→ script-kiddie) solo el **instructor**. El perfil usado se anota en la
bitácora/reporte.

## Configuration

**No requiere configuración.** Es un power de conocimiento: no tiene `mcp.json` ni API keys.
