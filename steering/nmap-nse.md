# Nmap Scripting Engine (NSE)

> Steering del power **footprinting-enumeration** — HTB *Network Enumeration with Nmap*, sección 7.

NSE permite interactuar con servicios mediante scripts en Lua. Los scripts se agrupan en 14 categorías.

| Categoría | Descripción |
|---|---|
| `auth` | Determinación de credenciales de autenticación. |
| `broadcast` | Descubrimiento por broadcast; los hosts hallados pueden añadirse a los escaneos. |
| `brute` | Fuerza bruta de login contra el servicio. |
| `default` | Los que ejecuta `-sC`. |
| `discovery` | Evaluación de servicios accesibles. |
| `dos` | Prueban denial of service; **dañan** el servicio, se usan poco. |
| `exploit` | Explotan vulnerabilidades conocidas del puerto. |
| `external` | Usan servicios externos para procesar. |
| `fuzzer` | Envían campos variados; lentos. |
| `intrusive` | Pueden afectar negativamente al objetivo. |
| `malware` | Comprueban si el sistema está infectado. |
| `safe` | Defensivos, no intrusivos ni destructivos. |
| `version` | Extensión de la detección de servicios. |
| `vuln` | Identifican vulnerabilidades concretas. |

> **Encuadre de enumeración:** en esta fase prioriza `default`, `safe`, `discovery` y `version`. Evita
> `dos`, `exploit`, `brute` e `intrusive` salvo autorización explícita del alcance: son ruidosos y
> pueden tumbar o alterar el servicio.

## Formas de invocar scripts

```bash
sudo nmap <IP> -sC                                   # categoría default (seguros)
sudo nmap <IP> --script <categoria>                  # toda una categoría
sudo nmap <IP> --script <script1>,<script2>          # scripts concretos
```

Listar los scripts disponibles de un servicio:

```bash
ls /usr/share/nmap/scripts/ | grep <servicio>        # ftp, smb, nfs, mysql, smtp...
```

## Ejemplos del módulo

```bash
# SMTP: banner + comandos permitidos (pueden revelar usuarios, VRFY, STARTTLS...)
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

# Escaneo agresivo: -sV + -O + --traceroute + scripts default, todo junto
sudo nmap 10.129.2.28 -p 80 -A

# Categoría vuln sobre HTTP: http-enum, vulners (CVEs), wordpress-users...
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

El `-A` reveló servidor (Apache 2.4.29), aplicación (WordPress 5.3.4), título del sitio y una
estimación de SO (Linux ~96%). Recuerda: el resultado de un NSE es una **pista**, no una prueba;
verifícalo con la herramienta nativa del servicio.

> **OS guesses y `vuln`:** las conjeturas de SO y los CVE que lista `vulners` son candidatos a
> investigar, no hechos confirmados. Anota el CPE/CVE y valida manualmente antes de afirmar nada.

Documentación de scripts: https://nmap.org/nsedoc/index.html
