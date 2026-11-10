---
name: footprinting
description: Conocimiento completo del módulo Footprinting de HTB Academy (21 secciones) para enumerar la presencia en Internet y los servicios de infraestructura de un objetivo (FTP, SMB, NFS, DNS, SMTP, IMAP/POP3, SNMP, MySQL, MSSQL, Oracle TNS, IPMI, gestión remota Linux/Windows) con nmap y OSINT, bajo metodología OODA. Enseña a descubrir y documentar, no a explotar. Use when el usuario pida escanear, enumerar, hacer footprinting/reconocimiento, usar nmap, o trabajar servicios de infraestructura en un laboratorio autorizado.
---

# Footprinting & Enumeration

Este skill entrega el **conocimiento completo del módulo Footprinting de HTB Academy (21
secciones)** para la fase de enumeración de una evaluación de seguridad **autorizada**:
descubrir y documentar la presencia en Internet de una organización y los servicios de
infraestructura que expone, sus versiones, banners y configuraciones inseguras.

La enumeración **no busca entrar** al sistema: busca **encontrar y entender todas las vías**
de acceso y dejarlas documentadas con evidencia. Un buen reporte de enumeración vale más que
un intento prematuro de fuerza bruta (ruidoso, provoca bloqueos y cierra puertas).

> **Regla de encuadre:** solo se enumeran activos dentro del alcance autorizado. Este material
> es didáctico; no concede autorización por sí mismo. El docente aprueba el alcance y cualquier
> acción de mayor impacto. En la enumeración pasiva (OSINT) no se tocan sistemas de terceros.

## Cómo está organizado el conocimiento (references/)

Las 21 secciones están repartidas en archivos de `references/` que cargas **bajo demanda**
(divulgación progresiva): lee solo el que necesites según la fase o el servicio.

| Reference | Secciones HTB | Contenido |
|---|---|---|
| `references/nmap.md` | (transversal) | Dominio de nmap: escaneo por fases, flags, NSE, timing, salida. Léelo primero. |
|  |  | *Para el flujo de nmap por fase en profundidad, el power incluye además guías de steering `nmap-*.md` (descubrimiento de hosts, escaneo de puertos, detección de servicios, NSE, rendimiento, evasión de firewall/IDS, salida/reportes), cargables bajo demanda.* |
| `references/osint.md` | 3, 4, 5 | Recon **pasivo**: Domain Information, Cloud Resources, Staff. Capa 1. |
| `references/servicios.md` | 6–12 | FTP, SMB, NFS, DNS, SMTP, IMAP/POP3, SNMP. |
| `references/bases-de-datos.md` | 13, 14, 15 | MySQL, MSSQL, Oracle TNS. |
| `references/gestion-remota.md` | 16, 17, 18 | IPMI, gestión remota Linux (SSH, Rsync, R-services) y Windows (RDP, WinRM, WMI). |
| `references/labs.md` | 19, 20, 21 | Enfoque metodológico de los labs Easy / Medium / Hard (sin spoilers). |

Las secciones 1 (Principios) y 2 (Metodología) están en este documento.

## Sección 1 — Principios de enumeración

El objetivo **no** es entrar a los sistemas, sino **encontrar todas las vías** para hacerlo.
La fuerza bruta prematura es ruidosa y provoca bloqueos (blacklisting), impidiendo seguir
probando. Analogía: un cazador de tesoros planifica y estudia mapas; no cava hoyos al azar.

**Los 3 principios:**
1. Hay más de lo que salta a la vista. Considera todos los puntos de vista.
2. Distingue entre lo que ves y lo que no ves.
3. Siempre hay formas de obtener más información. Entiende el objetivo.

> Cuando no sabemos cómo continuar, suele faltar **comprensión técnica**, no habilidad de
> pentesting. La tarea central no es explotar, sino descubrir **cómo** se puede explotar.

## Sección 2 — Metodología (6 capas / 3 niveles)

| Nivel | # | Capa | Categorías de información |
|---|---|---|---|
| **Infraestructura** | 1 | Internet Presence | Dominios, subdominios, vHosts, ASN, netblocks, IPs, cloud, medidas de seguridad |
| | 2 | Gateway | Firewalls, DMZ, IPS/IDS, EDR, proxies, NAC, segmentación, VPN, Cloudflare |
| **Host** | 3 | Accessible Services | Tipo de servicio, funcionalidad, config, puerto, versión, interfaz |
| | 4 | Processes | PID, datos procesados, tareas, origen, destino |
| **OS** | 5 | Privileges | Grupos, usuarios, permisos, restricciones, entorno |
| | 6 | OS Setup | Tipo de SO, parches, config de red, archivos de config, archivos privados sensibles |

Las capas 1 y 2 **no aplican** a redes internas tipo Active Directory. La metodología es un
conjunto de procedimientos sistemáticos, no una guía paso a paso; las herramientas y comandos
son un cheat sheet aparte.

## Metodología cognitiva: enumerar con OODA

| Etapa | Pregunta | Qué haces en enumeración |
|---|---|---|
| **Observe** | ¿Qué sé realmente? | Lee la salida cruda. Separa **hechos** (puerto 21 abierto) de **inferencias** (parece vsFTPd). |
| **Orient** | ¿Qué significa? | Relaciona banner/versión con config por defecto y ajustes peligrosos. Formula hipótesis. |
| **Decide** | ¿Qué conviene y está autorizado? | Elige el experimento **menos intrusivo** que reduzca la incertidumbre. Confirma alcance. |
| **Act** | ¿Qué pasó? | Ejecuta solo lo permitido, guarda la salida original, y vuelve a Observe. |

### Reglas cognitivas (críticas para el alumno)

1. **Un banner no es una vulnerabilidad.** Un puerto abierto prueba que un servicio respondió;
   un banner *sugiere* software/versión, pero no demuestra una CVE.
2. **Registra por separado** hechos observados, inferencias, hipótesis y hallazgos validados.
3. **Toda observación lleva procedencia** (fuente, comando, fecha) y **evidencia** (salida).
4. **No confíes ciegamente en herramientas automáticas**: dan falsos positivos (NSE
   `mysql-empty-password`, `VRFY` código 252). Verifica manualmente.
5. **Después de actuar, revisa**: terminar sin error no significa éxito.
6. **La información de un servicio alimenta a otro.** Encadenar es el corazón del módulo.

## Enumeración activa vs. pasiva

- **Pasiva (OSINT, secciones 3–5):** terceros y fuentes públicas; navegas como "visitante" sin
  tocar los sistemas del objetivo. No requiere escaneo. No se prueban activos de terceros.
- **Activa (secciones 6–18):** escaneas e interrogas directamente los servicios del objetivo
  dentro del alcance. Empieza **siempre** por nmap (ver `references/nmap.md`).

## nmap: el punto de partida (resumen)

```bash
export TARGET=<ip-autorizada>
nmap -sV -sC -oA evidencia/nmap-inicial $TARGET   # versión + NSE por defecto, guarda evidencia
sudo nmap -p- -sV $TARGET                          # todos los puertos TCP
sudo nmap -sU -p53,111,161 -sV $TARGET             # UDP relevante (DNS, rpcbind, SNMP)
```

| Flag | Significado |
|---|---|
| `-sV` | Detección de versión del servicio |
| `-sC` | Scripts NSE por defecto (seguros) |
| `-A` | Agresivo: OS + versión + scripts + traceroute |
| `-p` / `-p-` | Puertos concretos / todos los 65535 TCP |
| `-sU` | Escaneo UDP |
| `--script <nombre>` | NSE específicos (`smb-*`, `nfs*`, `mysql*`, `oracle-sid-brute`...) |
| `--open` | Solo puertos abiertos |
| `-oA <base>` | Guardar en los 3 formatos (normal, grepable, XML) |

## Resumen de puertos del módulo

| Puerto | Servicio | Sección |
|---|---|---|
| 21 / 20 | FTP (control / datos) | 6 |
| 22 | SSH | 17 |
| 25 / 465 / 587 | SMTP / SMTPS / Submission | 10 |
| 53 | DNS | 9 |
| 110 / 995 | POP3 / POP3S | 11 |
| 111 / 2049 | NFS (rpcbind / nfs) | 8 |
| 135 | WMI/RPC (Windows) | 18 |
| 139 / 445 | SMB (NetBIOS / CIFS) | 7 |
| 143 / 993 | IMAP / IMAPS | 11 |
| 161 / 162 | SNMP / traps (UDP) | 12 |
| 512 / 513 / 514 | R-services | 17 |
| 623 | IPMI (UDP) | 16 |
| 873 | Rsync | 17 |
| 1433 / 1434 | MSSQL / DAC | 14 |
| 1521 | Oracle TNS | 15 |
| 3306 | MySQL | 13 |
| 3389 | RDP | 18 |
| 5985 / 5986 | WinRM (HTTP / HTTPS) | 18 |

## Evidencia y documentación

> **Regla de persistencia de evidencia (obligatoria):** la **evidencia cruda SÍ se guarda** y
> se conserva. **Nunca** la dejes solo en rutas temporales (`/tmp`, `/var/tmp`, o el `/tmp` de un
> contenedor): esas ubicaciones se borran y la evidencia se pierde. Guárdala **en el workspace del
> proyecto**, en un directorio `evidencia/` por evaluación, versionable y accesible. Si una
> herramienta escribe primero en `/tmp` (p. ej. dentro de un contenedor de ataque), **cópiala al
> workspace al terminar** y solo entonces limpia la copia temporal.

Para cada servicio registra, en el directorio `evidencia/` de la evaluación (en el workspace):
**mapa de servicios** (puerto → servicio → versión → banner), la **salida original** de cada
comando (`-oA` en nmap; `tee` en el resto), los **hallazgos** clasificados (observación /
hipótesis / confirmado), la **información encadenada** (usuarios, credenciales, hosts internos,
hashes), las **flags** (`FLAG{...}`) y las **recomendaciones** de remediación por cada
misconfiguración.

## Preguntas didácticas (úsalas en cada servicio)

- ¿Qué afirmación **observaste** y cuál solo **inferiste**?
- ¿Qué **fuente/evidencia** respalda tu conclusión?
- ¿Qué evidencia adicional **confirmaría o refutaría** la hipótesis?
- ¿Por qué elegiste esta herramienta y no una **menos intrusiva**?
- ¿La acción está dentro del **alcance aprobado**?
- ¿Cómo explicarías el **riesgo** al equipo técnico y a la dirección sin tecnicismos?

## Referencias

- HTB Academy — módulo *Footprinting* (21 secciones), PDFs en `footprinting/footprinting_htb/`.
- Material local: `footprinting/apuntes_footprinting.md`, `footprinting/cheatsheet_comandos.md`,
  `footprinting/labfootprinting/guia-del-alumno.md`.
