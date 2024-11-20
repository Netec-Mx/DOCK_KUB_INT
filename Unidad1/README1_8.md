# Práctica 1.8 Contenedor Utilitario

## Objetivo
Al finalizar esta práctica, serás capaz de crear y utilizar un contenedor Docker utilitario con un cliente ligero para conectarte a la base de datos del microservicio ms-productos utilizando sqlplus.

## Duración

15 minutos

## Instrucciones

### Paso 1. Verificar el contenedor Oracle

Antes de proceder, asegúrate de que el contenedor Oracle llamado `dki-oradb` está en ejecución y accesible. 

1. Lista los contenedores activos:

```cmd
docker ps
```

2. Verifica que el contenedor `dki-oradb` está en la lista con el puerto mapeado, por ejemplo, 1521 para Oracle DB.

<br/>

### Paso 2. Crear un contenedor utilitario con sqlplus

Vamos a usar una imagen ligera de Oracle Instant Client para conectarnos a la base de datos.

1. Ejecuta el siguiente comando para lanzar un contenedor interactivo:

```cmd
docker run -it --rm --network=dki-network ghcr.io/oracle/oraclelinux8-instantclient:19 sqlplus SYSTEM/Netec_123@//dki-oradb:1521/XE
``

**Nota**: La opción `--network host`  asegura que el contenedor utilitario pueda comunicarse con el contenedor de Oracle Database si estás en un entorno local. Si estás en otro entorno de red, ajusta las configuraciones de red según sea necesario.

<br/>

### Paso 3. Instalar Oracle Instant Client dentro del contenedor

Dentro del contenedor interactivo, ejecuta los siguientes comandos:

1. Verifica que sqlplus está instalado:

```cmd
sqlplus -version

lsnrctl status
```

<br/>

### Paso 4. Conexión a la base de datos Oracle Dockerizada

Usa `sqlplus` para conectarte al contenedor `dki-oradb`.

1. Conecta usando el siguiente comando, ajustando los valores según tu configuración (usuario, contraseña y servicio):

```cmd

sqlplus USERNAME/PASSWORD@HOST:PORT/SERVICENAME
```

Ejemplo (ajustar según sea necesario):

```cmd
sqlplus sys/Netec_123d@localhost:1521/XE
```

2. Una vez conectado, puedes consultar la tabla creada por el microservicio `ms-productos`. Por ejemplo:

```sql

show user;

show con_name;

column name format a10

SELECT con_id, name, open_mode from v$pdbs;

-- Debería de aparecer el usuario dkuser

SELECT username FROM dba_users order by 1;

SELECT table_name FROM all_tables WHERE owner = 'DKUSER';

desc dkuser.productos;

SELECT * FROM dkuser.productos;

SELECT count(*) from dkuser.productos;

alter session set contaiener=xepdb1

```

### Paso 5. Salir del contenedor
Cuando termines, sal del contenedor utilitario escribiendo:

```sql

exit
```

### Notas Adicionales

- Si necesitas un alias para simplificar la conexión con sqlplus, puedes configurarlo en el archivo **tnsnames.ora**. Esto puede hacerse en un paso avanzado si decides persistir la configuración del contenedor utilitario.

- Asegúrate de revisar que los puertos y credenciales sean los correctos antes de intentar conectarte.

<br/> <br/>

## Resultado Esperado