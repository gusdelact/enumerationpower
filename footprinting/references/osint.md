# OSINT y reconocimiento pasivo (secciones 3–5, capa 1)

Recolección **pasiva**: navegas como "cliente/visitante" usando terceros y fuentes públicas,
sin escaneos activos ni conexiones directas que expongan tu actividad. Cubre la capa 1
(Internet Presence) de la metodología. **No pruebes activos de terceros sin permiso.**

> OODA: aquí observas la presencia pública de la organización y la orientas para construir un
> inventario (subdominios, IPs, proveedores, personas) que alimentará la enumeración activa.

---

## Sección 3 — Domain Information

Combina el **primer y segundo principio**: analiza lo que ves (los servicios de la web
principal) y deduce lo que *no* ves (las tecnologías necesarias para ofrecerlos). Toma la
perspectiva del desarrollador.

### Web principal y certificado SSL
- Lee la web: qué servicios ofrece y qué tecnologías/estructuras requieren.
- El **certificado SSL** suele cubrir varios subdominios, probablemente activos.

### crt.sh — Certificate Transparency (RFC 6962)
Toda CA registra los certificados emitidos en logs auditables. Útil para descubrir subdominios.

```bash
# Todos los registros en JSON
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq .

# Solo subdominios únicos
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" \
  | jq . | grep name | cut -d":" -f2 | grep -v "CN=" \
  | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

### Resolver subdominios propios y construir lista de IPs
Solo hosts de la propia empresa (no de terceros, que no puedes testear sin permiso):

```bash
# Subdominio -> IP (filtrando los de la empresa)
for i in $(cat subdomainlist); do \
  host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4; done

# Construir lista de IPs
for i in $(cat subdomainlist); do \
  host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt; done
```

### Shodan
Dispositivos permanentemente conectados a Internet (IoT, cámaras, servidores, controladores
industriales). Revela puertos abiertos y servicios (HTTP/S, FTP, SSH, SNMP, Telnet, RTSP, SIP).

```bash
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

### Registros DNS (`dig any`)

```bash
dig any inlanefreight.com
```

| Registro | Qué revela |
|---|---|
| **A** | IPv4 del (sub)dominio. |
| **MX** | Servidores de correo responsables (p. ej. Google → saltar por ahora). |
| **NS** | Name servers → identifican al proveedor de hosting. |
| **TXT** | Claves de verificación de terceros + SPF/DMARC/DKIM. **Muy valioso.** |
| **SOA** | Contacto administrativo (el `.` del email se reemplaza por `@`). |

**Interpretación de proveedores en TXT** (pistas de infraestructura):
- **Atlassian** → desarrollo/colaboración (posibles recursos accesibles; Confluence, Jira).
- **Google/Gmail** → gestión de correo; posible GDrive abierto por link.
- **LogMeIn** → acceso remoto centralizado (comprometerlo = acceso total; ojo reutilización).
- **Mailgun** → APIs de email → probar IDOR, SSRF, PUT/POST en interfaces.
- **Outlook/O365** → OneDrive y Azure blob/file storage (¡usa SMB!).
- **INWX** → proveedor de hosting; el TXT `MS=` suele parecerse al usuario/ID de gestión.
- En **SPF** aparecen a veces **IPs internas** (`ip4:10.x.x.x`): anótalas.

---

## Sección 4 — Cloud Resources

AWS (**S3 buckets**), Azure (**blobs**), GCP (**cloud storage**). Mal configurados, son
accesibles **sin autenticación**.

- El storage suele aparecer en DNS (p. ej. `s3-website-us-west-2.amazonaws.com`).
- **Descubrimiento pasivo:**
  - **Google Dorks**: `inurl:`, `intext:` + nombre de empresa (encuentra PDFs, docs, código).
  - El storage también aparece en el **código fuente** de la web (imágenes, JS, CSS).
  - **domain.glass**: infraestructura y si hay **Cloudflare** (= medida de seguridad, capa 2).
  - **GrayHatWarfare**: busca buckets AWS/Azure/GCP y filtra por formato de archivo.
- Las empresas usan **abreviaturas** de su nombre en IT → útil para adivinar nombres de storage.
- Riesgo crítico: **claves SSH privadas filtradas**, descargables por cualquiera → login sin
  contraseña a una o varias máquinas.

---

## Sección 5 — Staff

LinkedIn / Xing y **ofertas de empleo** revelan tecnologías, lenguajes, frameworks, bases de
datos y medidas de seguridad.

- Ejemplo real de job post: Java/C#/C++, Python/Ruby/PHP/Perl, PostgreSQL/MySQL/SQL Server/
  Oracle, ORM (SQLAlchemy, Hibernate, Entity Framework), Flask/Django/Spring/ASP.NET, Git/SVN/
  Perforce, Atlassian Suite, Docker/Kubernetes, Redis.
- Perfiles y **repos públicos** de empleados pueden filtrar emails personales, **tokens JWT
  hardcodeados**, claves y rutas de archivos (muchos copian nombres de archivos de guías/best
  practices, p. ej. OWASP Top 10 para Django).
- Para mapear infraestructura: busca empleados **técnicos de desarrollo y de seguridad**; el
  área de seguridad delata qué controles tiene la empresa.

**Documenta (OSINT):** inventario de subdominios, IPs propias, proveedores detectados (con su
interpretación), recursos cloud públicos, posibles credenciales/claves filtradas y perfil
tecnológico del objetivo. Todo con su fuente y fecha. Esto alimenta la enumeración activa.
