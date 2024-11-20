# Práctica 1.7 Consumo de Microservicios - Caso de Estudio

## Objetivo
Al finalizar esta práctica, serás capaz de consumir microservicios mediante el uso de herramientas como Postman o implementaciones en código.  

## Duración
15 minutos

<br/>

## Instrucciones 

### Consumo con el comando curl del microservicio ms-productos

En esta primera parte, utilizarás comandos **curl** para consumir los endpoints del microservicio `ms-productos`. Sigue los pasos detallados para probar cada operación (GET, POST, PUT, DELETE).

<br/>

### 1. Endpoint: Listar Todos los Productos

Realiza una solicitud para obtener todos los productos registrados.

```cmd
 
curl -X GET http://localhost:9081/productos
```

- Verifica que el microservicio retorna una lista de productos (puede estar vacía si no se han creado registros).

<br/>

### 2. Endpoint: Obtener Producto por ID

Consulta un producto específico por su identificador.

```cmd
 
curl -X GET http://localhost:9081/productos/{id}
```

**Nota**: Reemplaza {id} con el ID de un producto existente.

- Comprueba que el servicio retorna el producto correspondiente al ID proporcionado o un error 404 si no existe.

<br/>

### 3. Endpoint: Crear un Producto

Envía una solicitud para crear un nuevo producto.

```cmd
 
curl -X POST http://localhost:9081/productos -H "Content-Type: application/json" -d '{ "nombre": "Laptop Dell XPS 15",
  "descripcion": "Laptop de alta gama con procesador Intel Core i7, 16GB RAM y 512GB SSD.", "precio": 1500.99, "stock": 10 }'

```
- Verifica que el producto se crea correctamente y que se retorna el producto creado con un ID generado.

<br/>

### 4. Endpoint: Actualizar un Producto

Actualiza los datos de un producto existente.

```cmd
 
curl -X PUT http://localhost:9081/productos/{id} -H "Content-Type: application/json" -d {
  "nombre": "Laptop Dell XPS 15", "descripcion": "Laptop de alta gama con procesador Intel Core i9, 32GB RAM y 2TB SSD." , "precio": 150000.99, "stock": 5 }'
```

**Nota**: Reemplaza {id} con el ID de un producto existente.

- Asegúrate de que el producto se actualiza correctamente y de que el servicio retorna el producto actualizado.

<br/>

### 5. Endpoint: Eliminar un Producto

Elimina un producto por su identificador.

```cmd
curl -X DELETE http://localhost:9081/productos/{id}
```

**Nota**: Reemplaza {id} con el ID de un producto existente.

- Confirma que el producto se elimina correctamente y que el servicio retorna un estado 204 (No Content).

<br/>

### 6. Verificación Final

- Después de probar los endpoints, vuelve a ejecutar el comando para "Listar Todos los Productos" y confirma los cambios (nuevos productos, actualizaciones o eliminaciones).

```cmd
 
curl -X GET http://localhost:9081/productos
```

<br/>

### Entrega y Observaciones

- Documenta los resultados de cada comando curl, indicando los datos enviados y las respuestas obtenidas.
- Responde a las siguientes preguntas:
    - ¿Qué estado HTTP retorna cada endpoint?
    - ¿Los datos enviados y recibidos coinciden con lo esperado?
    - ¿Qué ocurre si consultas un producto con un ID inexistente?

<br/>
<br/>

## Resultado Esperado