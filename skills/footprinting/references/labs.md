# Footprinting Labs — enfoque metodológico (secciones 19–21)

Los tres labs finales del módulo consolidan todo lo anterior sobre objetivos reales. Esta guía
da el **escenario, las pistas y el método**; no una solución paso a paso. El valor pedagógico
está en que el alumno **razone y encadene** la información con OODA, no en entregarle la flag.

## Escenario común

La empresa **Inlanefreight Ltd** contrata un pentest de **3 servidores internos**. Está
**prohibido** usar exploits agresivos: son sistemas de **producción**. El trabajo es de
**enumeración**: identificar servicios, versiones, configuraciones inseguras y encadenar la
información (credenciales, usuarios, claves SSH, hashes) reutilizándola entre servicios.

> Regla: cada lab es un activo autorizado del escenario. Mantén el escaneo controlado (sin
> exploits agresivos) tal como exige el enunciado.

## Método recomendado (igual para los tres)

1. **nmap primero**: `nmap -sV -sC -oA evidencia/<lab> $TARGET` y luego `-p-` para no perder
   servicios. Construye el mapa de servicios antes de decidir por dónde entrar.
2. **Orienta por servicio**: para cada puerto abierto, abre el playbook correspondiente
   (`servicios.md`, `bases-de-datos.md`, `gestion-remota.md`) y revisa configs por defecto y
   ajustes peligrosos.
3. **Enumera lo accesible sin credenciales** (anónimo, null session, AXFR, VRFY, exports NFS).
4. **Encadena**: toda credencial, usuario, clave o host interno que encuentres pruébalo en los
   demás servicios (reutilización de contraseñas es la clave del módulo).
5. **Documenta y para** cuando tengas la evidencia del objetivo. No escales con exploits.

## Pistas por lab (del enunciado del módulo)

| Lab | Servidor | Pistas del enunciado | Objetivo |
|---|---|---|---|
| **Easy** (19) | DNS interno | Credenciales `ceil:qwer1234`; empleados mencionaban **claves SSH** en un foro | Contenido de `flag.txt` |
| **Medium** (20) | Accesible desde toda la red interna | — | Password del usuario `HTB` |
| **Hard** (21) | MX + gestión; hay backup de cuentas del dominio | — | Password del usuario `HTB` |

## Preguntas que el alumno debe poder responder al entregar

- ¿Qué servicios expone cada servidor y con qué versión? (tabla con evidencia)
- ¿Qué misconfiguración permitió el acceso inicial, y por qué es peligrosa?
- ¿Cómo encadenaste la información de un servicio al siguiente?
- ¿Qué habría impedido este acceso (recomendación de remediación)?
- ¿Qué afirmaste como **confirmado** y qué quedó como **hipótesis**?

## Entregable

Un reporte de enumeración por lab con: mapa de servicios, hallazgos por servicio (clasificados),
cadena de información (credenciales/usuarios/claves/hosts), flags como evidencia y
recomendaciones de remediación. La flag es la *prueba* de enumeración completa, no el fin en sí.
