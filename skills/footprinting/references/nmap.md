# nmap — guía de escaneo para enumeración

nmap conduce toda la enumeración: produce el **mapa de servicios** (puertos, versiones,
banners, scripts) del que cuelga todo lo demás. Esta guía define los comandos por fases,
cómo controlar el ruido y cómo leer la salida. Reemplaza `$TARGET` por la IP del activo
**autorizado** (`export TARGET=<ip>`).

> Regla OODA: cada comando es un **Act**. Antes, decide el menos intrusivo que resuelva tu
> duda; después, vuelve a **Observe** y lee la salida cruda separando hechos de inferencias.

---

## Fase 0 — Preparación

```bash
export TARGET=<ip-autorizada>
mkdir -p evidencia                 # salidas originales van aquí
```

Guarda **siempre** la salida con `-oA`, que escribe los tres formatos a la vez:

```bash
nmap -sV -sC -oA evidencia/nmap-inicial $TARGET
#   -> evidencia/nmap-inicial.nmap  (legible)
#   -> evidencia/nmap-inicial.gnmap (grepable)
#   -> evidencia/nmap-inicial.xml   (XML, para informes/parseo)
```

---

## Fase 1 — Descubrimiento de servicios (lo primero, siempre)

```bash
# Barrido estándar: versión de servicio + scripts NSE por defecto (seguros)
nmap -sV -sC $TARGET
```

- `-sV` interroga cada puerto para deducir **servicio y versión** (banner grabbing dirigido).
- `-sC` ejecuta la categoría `default` de scripts NSE: segura y poco intrusiva.

**Observa y anota:** ¿qué puertos abiertos hay? ¿qué servicio y versión reporta cada uno?
En el lab deberías ver aproximadamente: 21, 25, 53, 110, 111, 139, 143, 445, 993, 995, 2049,
3306 (más el rango pasivo de FTP en puertos altos).

---

## Fase 2 — Cobertura completa (no te pierdas nada)

```bash
# TODOS los puertos TCP (los 65535). Más lento; hazlo para no dejar servicios ocultos.
sudo nmap -p- -sV $TARGET

# Solo puertos abiertos, salida limpia
sudo nmap -p- --open $TARGET

# UDP relevante para este lab: DNS (53) y rpcbind (111)
sudo nmap -sU -p53,111 -sV $TARGET
```

`sudo` es necesario para escaneos SYN (`-sS`, por defecto con privilegios) y para UDP (`-sU`).
UDP es lento y propenso a `open|filtered`; limítalo a los puertos que te interesan.

---

## Fase 3 — Enfoque por servicio con NSE

Cuando ya sabes qué hay, profundiza con scripts específicos. Lista de NSE disponibles:

```bash
ls /usr/share/nmap/scripts/ | grep <servicio>      # p. ej. ftp, smb, nfs, mysql
```

Comandos NSE por servicio del lab:

```bash
# FTP (21)
sudo nmap -sV -p21 -sC -A $TARGET

# SMB (139/445)
sudo nmap -sV -sC -p139,445 $TARGET

# NFS (111/2049)
sudo nmap -sV -p111,2049 --script nfs* $TARGET

# DNS (53)
sudo nmap -sV -sC -p53 $TARGET

# SMTP (25)
sudo nmap -sC -sV -p25 $TARGET
sudo nmap -p25 --script smtp-open-relay -v $TARGET

# IMAP/POP3 (110,143,993,995) — lee el certificado (org, FQDN, email admin)
sudo nmap -sV -sC -p110,143,993,995 $TARGET

# MySQL (3306) — ojo: mysql-empty-password da falsos positivos, verifica a mano
sudo nmap -sV -sC -p3306 --script mysql* $TARGET
```

---

## Control de ruido y timing

```bash
# Plantillas de tiempo: -T0 (sigiloso/lento) ... -T5 (muy rápido/ruidoso). -T3 es el defecto.
nmap -sV -sC -T3 $TARGET

# Host que no responde a ping: trátalo como vivo
nmap -Pn $TARGET

# Sin ARP ping y sin resolución DNS (evita tráfico/ruido extra)
nmap --disable-arp-ping -n $TARGET
```

En redes de producción/laboratorio: evita `-T5` y escaneos agresivos de exploits. El objetivo
de la enumeración es descubrir, no tumbar servicios.

---

## Depuración

```bash
# Ver los paquetes que nmap envía/recibe (entender qué pasa en la red)
sudo nmap -sV -p21 --packet-trace $TARGET

# Trazar la ejecución de los scripts NSE
sudo nmap -sV -p21 -sC --script-trace $TARGET

# Verbosidad / razón de cada estado de puerto
nmap -v -sV --reason $TARGET
```

---

## Cómo leer la salida (Observe)

- **`open`**: el servicio respondió. Es un **hecho**.
- **`filtered`**: un firewall probablemente descarta los paquetes; no confundas con cerrado.
- **`open|filtered`** (típico en UDP): no hubo respuesta concluyente.
- **Versión/banner**: es una **inferencia** sobre el software; puede estar falseado o ser
  aproximado. No lo eleves a "vulnerabilidad confirmada".
- **Resultado de NSE**: una **pista**, no una prueba. Verifícalo manualmente con la herramienta
  nativa del servicio (ver `servicios.md`).

### Registro mínimo por puerto

| Puerto | Estado | Servicio | Versión (inferida) | Script/observación | Evidencia |
|---|---|---|---|---|---|
| 21/tcp | open | ftp | vsftpd 3.x | ftp-anon: Anonymous login allowed | nmap-inicial.nmap |

Lleva esta tabla a tu reporte y enlázala con el archivo de salida original en `evidencia/`.
