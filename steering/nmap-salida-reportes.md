# Nmap: guardar resultados y generar reportes

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 5.

Guarda **cada** escaneo: sirve para comparar métodos, para documentar y para el reporte final. Nmap
escribe en tres formatos y uno que los agrupa.

| Opción | Formato | Extensión | Para qué |
|---|---|---|---|
| `-oN` | Normal | `.nmap` | Lectura humana directa. |
| `-oG` | Grepable | `.gnmap` | Filtrar con `grep`/`cut`/`awk`. |
| `-oX` | XML | `.xml` | Parseo e informes (XSLT, herramientas). |
| `-oA <base>` | Los tres a la vez | | Lo habitual: no pierdes ningún formato. |

```bash
sudo nmap 10.129.2.28 -p- -oA evidencia/target
# genera evidencia/target.nmap, target.gnmap y target.xml
```

Si no das ruta completa, se guarda en el directorio actual.

> **Regla de persistencia de evidencia:** guarda siempre en el directorio `evidencia/` del **workspace
> del proyecto**, versionable. Nunca dejes la salida solo en `/tmp` (se borra). Si una herramienta
> escribe primero en `/tmp` (p. ej. dentro de un contenedor de ataque), **cópiala al workspace** al
> terminar y solo entonces limpia la copia temporal.

## El formato grepable en la práctica

```bash
cat evidencia/target.gnmap
# Host: 10.129.2.28 () Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, ...
```

Útil para extraer rápidamente puertos abiertos:

```bash
grep "/open/" evidencia/target.gnmap
```

## Reporte HTML a partir del XML

El XML se convierte en un HTML claro y legible, ideal para documentación y para audiencia no técnica:

```bash
xsltproc evidencia/target.xml -o evidencia/target.html
```

Ábrelo en el navegador: presenta puertos, servicios y estados de forma estructurada.

Más sobre formatos de salida: https://nmap.org/book/output.html
