# Práctica 2.3 Creación de un Archivo Docker Compose Intermedio

## Objetivo

Al finalizar esta práctica, serás capaz de crear un archivo docker-compose.yml de nivel intermedio que defina dos contenedores simples:

1. Un servicio NGINX que sirva en el puerto 9999.

2. Un servicio MongoDB con persistencia de datos mediante volúmenes

## Duración

20 minutos


## Material Necesario

- Docker y Docker Compose instalados y funcionando

- Un editor de texto (VSC, vim. Notepad++, etc.).

- Acceso a la terminal o consola del entorno de curso,


<br/>

## Instrucciones

### Paso 1. Configura tu entorno de trabajo

1. Crea una carpeta para almacenar los archvios de la práctica

```cmd
mkdir practica_2_3
cd practica_2_3
```

2. Crea un archivo vacío llamado `docker-compose.yml`

```cmd
touch docker-compose.yml
```

### Paso 2. Escribe el contenido del archivo `docker-compose.yml`

Abre el archivo en tu editor de texto de preferencia y escribe las instrucciones necesarias que debe de contener el archivo YAML.

1. Especifica que versión estás utilizando el esquema de Compose versión 3.9

2. Define dos servicios:

    - **nginx**: Define un contenedor basado en la imagen oficial `nginx:latest` y expone el puerto `9999`.

    - **mongodb**: Define un contenedor basado en la imagen oficial `mongo:latest` con persistencia de datos en el volumen `mongo-data` expuesto en el puerto `27017`.

3. Define un volumen para almacenar datos de MongoDB.

4. Permite la comunicación entre los servicios **nginx** y **mongodb**.

<br/>

### Paso 3. Valida el archivo

Antes de ejecutar el archivo, valida su sintaxis con el comando:

```cmd
docker-compose config
```

Si hay errores, revisa los mensajes proporcionados para corregirlos.

<br/>

### Paso 4. Ejecuta los servicios

1. Inicia los contenedores con:

```cmd
docker-compose up -d

docker ps

docker volume ls

docker network ls

```
<br/>

### Paso 5. Verifica el funcionamiento

1. Prueba el servicio NGINX:

    - Abre un navegador y accede a `http://localhost:9999`

    - Deberías ver la página de inicio de NGINX.

2. Prueba el servicio MongoDB:

    - Usa la herramienta de `mongosh` o un cliente MongoDB para conectarte al puerto `27017`

    ```cmd
    mongosh --host localhost --port 27017
    ```

    - Ejecuta el comando 

    ```javascript

    show dbs;

    db.stats();

    use my_db

    db.my_collection.insertOne({ name: "Producto 1", price: 100, stock: 50 });

    db.my_collection.find();

    db.my_collection.updateOne({ name: "Producto 1" }, { $set: { stock: 5 } })

    db.my_collection.find();

    db.my_collection.deleteOne({ name: "Producto 1" });

    db.stats();

    ```

<br/>

### Paso 7. Detén y elimina los contenedores Docker

Cuando termines, detén y elimina los recursos creados:

```cmd

docker-compose down --volumes

docker ps

docker volume ls

docker network ls

```
<br/>
<br/>

## Resultados Esperados

- Archivo `docker-compose.yml` funcional con dos servicios definidos.

- NGINX sirviendo en el puerto `9999`.

- MongoDB funcionando con persistencia de datos en el volumen `mongo-data`.

- Conocimiento práctico sobre cómo definir servicios, volúmenes y redes en Docker Compose.