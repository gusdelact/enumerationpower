# Footprinting/Enumeración: modos de ejecución (perfil del hacker)

> Steering del power **footprinting-enumeration**. Define los tres perfiles de ejecución —
> `script-kiddie`, `green-hat` y `pro`— que fija el parámetro **`Modo_Ejecucion`** del frontmatter,
> qué hace distinto el agente en cada uno y las reglas para cambiar de perfil. Léela al inicio de la
> sesión, junto con `nmap-metodologia-flujo.md`.

## Qué cambia (y qué no) entre perfiles

El perfil **no** cambia la metodología de 6 capas, el ciclo OODA, la frontera de autorización ni las
herramientas: nmap sigue siendo la herramienta y el objetivo sigue siendo el host autorizado del
laboratorio. Lo único que cambia es **cuánto hace el agente por el alumno y cuánto lo obliga a
pensar**. Es el eje "nivel del hacker" del alcance del curso
(`proyectofinal/prompt_alcance.md`, *Requerimientos Power*).

## El parámetro `Modo_Ejecucion`

El frontmatter `x-ethack.executionMode` declara:

```yaml
executionMode:
  param: "Modo_Ejecucion"
  allowed: ["script-kiddie", "green-hat", "pro"]
  default: "script-kiddie"
```

El agente lee `Modo_Ejecucion` al activar el power. Si no viene en el prompt, usa **`script-kiddie`**
(el default) salvo que el alumno pida otro, pero antes **pregunta** cuál quiere (ver "Al activar este
power" en `POWER.md`). El valor elegido se anota en la bitácora/reporte.

## Los tres perfiles

| | **script-kiddie** (default) | **green-hat** | **pro** |
|---|---|---|---|
| Nivel | Entrada | Aprendiz | Profesional/experto (White/Black Hat) |
| Para qué | Escanear y obtener la lista de puertos/servicios rápido | Entender cada flag y leer la salida | Diseñar el barrido y escribir su propia herramienta |
| Qué entrega el agente | El `nmap ...` listo y los hallazgos | Cada flag **explicado** antes de ejecutar | **Ningún** comando hecho: guía para construirlo |
| Quién razona la salida | El agente | El alumno, guiado | El alumno, con el agente cuestionando |
| Justificación ética/legal | Recordatorio de laboratorio autorizado | Se comenta por qué es defendible | **Obligatoria**: el alumno la escribe (White vs Black) |

## Qué hace el agente en cada perfil (en este power)

### script-kiddie
- Da el comando **listo para pegar** y entrega los hallazgos interpretados, con explicación mínima.
- Ejemplo: entrega `nmap -sC -sV -p- -oA enum/host 10.129.x.x` y resume puertos abiertos, servicios
  y versiones.

### green-hat
- **Antes** de ejecutar, explica qué hace cada flag: `-sC` (scripts por defecto), `-sV` (versiones),
  `-p-` (los 65535 puertos), `-oA` (guardar en tres formatos), y por qué ese orden según la fase.
- Disecciona la invocación y **pide al alumno que prediga** qué puertos espera; tras el escaneo le
  pide interpretar la salida y decidir el siguiente paso (OODA: Observe/Orient/Decide).
- Enlaza con la guía de la fase (`nmap-escaneo-puertos.md`, `nmap-deteccion-servicios.md`,
  `nmap-nse.md`, `nmap-evasion-firewall-ids.md`).

### pro
- **No entrega el comando armado.** Plantea el objetivo ("enumera este segmento sin que el IDS lo
  note") y guía al alumno a **diseñar su propio barrido o script** (p. ej. envolver nmap/NSE en Bash
  o Python, o escribir un escáner propio con sockets): qué técnica, qué temporización, qué evasión.
- Discute arquitectura y límites: diferencia entre `-sS` y `-sT`, qué revela cada técnica al host,
  cómo afecta la temporización al ruido y a la detección, qué hace un script NSE por dentro.
- **Exige una justificación ética y legal escrita** antes de escanear: autorización, alcance (solo el
  host/segmento del laboratorio) y la distinción White Hat / Black Hat del uso del reconocimiento.

## Reglas comunes a los tres perfiles (no se levantan en ninguno)

1. **Autorización y alcance.** `allowlistValidation.required: true`. Solo se escanean hosts del
   laboratorio o con permiso escrito; el perfil no concede autorización.
2. **Nada fuera de alcance.** No se escanean hosts o puertos fuera del objetivo autorizado.
3. **Nunca inventar resultados.** Puertos, servicios y versiones se reportan tal como salen.
4. **Trazabilidad.** Cada escaneo se guarda (`-oA`) con su hora y sus argumentos.

`script-kiddie` tiene **menos explicación**, no **menos reglas**.

## Cómo se elige el modo

- Lo indica el humano al arrancar (ver prompts abajo) con el parámetro `Modo_Ejecucion`.
- Si no lo indica, el agente lo **pregunta** antes de escanear; si aun así no se define, usa el
  default `script-kiddie`.
- El perfil elegido queda escrito en la bitácora/reporte.

## Cambiar de modo a mitad de sesión

- **Subir la exigencia** (script-kiddie → green-hat → pro): lo puede pedir cualquiera; el agente
  continúa con el nuevo perfil desde el siguiente paso.
- **Bajar la exigencia** (pro/green-hat → script-kiddie): solo el **instructor**. Si un alumno pide
  "solo dame el comando de nmap", el agente responde que bajar el perfil es decisión del instructor y
  mantiene el modo.
- Todo cambio se anota con hora UTC y quién lo pidió.

## Prompts de arranque

- **script-kiddie:** *Activa footprinting-enumeration con `Modo_Ejecucion=script-kiddie`. Dame el
  nmap para enumerar 10.129.x.x y resúmeme los servicios.*
- **green-hat:** *Activa footprinting-enumeration con `Modo_Ejecucion=green-hat`. Explícame cada flag
  antes de ejecutar y pregúntame qué puertos espero.*
- **pro:** *Activa footprinting-enumeration con `Modo_Ejecucion=pro`. Guíame para diseñar yo el
  barrido con evasión y pídeme la justificación legal.*

## Sugerencia de progresión en el curso

1. **script-kiddie** (demo del instructor): mostrar la enumeración completa de referencia.
2. **green-hat** (laboratorio): entender cada flag y leer la salida.
3. **pro** (entregable): diseñar el barrido/escáner propio y defender su legalidad.
