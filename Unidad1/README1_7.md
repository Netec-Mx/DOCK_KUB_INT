# Práctica 1.7 Consumo de Microservicios - Caso de Estudio

## Objetivo
Al finalizar esta práctica, serás capaz de consumir microservicios mediante el uso de herramientas como Postman o implementaciones en código.  

## Duración
20 minutos

<br/>

## Preparativos

1. Asegúrate de que el microservicio `ms-productos` esté en ejecución y accesible en el puerto 9081.

2. Abre Postman y opcionalmente crea una colección para organizar las solicitudes relacionadas con el microservicio.

3. Asegúrate de que el microservicio `ms-deseos` esté en ejecución y accesible en http://localhost:9084.

4. Abre Postman y opcionalmente crea una nueva colección llamada ms-deseos.

<br/>

## Instrucciones 

### 1. Obtener todos los productos

- Método HTTP: GET
- URL: http://localhost:9081/productos
- Descripción: Recupera una lista de todos los productos disponibles.
- Paso en Postman:
    1. Crea una nueva solicitud en la colección ms-productos con el método GET y la URL especificada.
    2. Ejecuta la solicitud y verifica que se devuelva una lista en formato JSON.

<br/>

### 2. Obtener un producto por ID
- Método HTTP: GET
- URL: http://localhost:9081/productos/{id}
- Reemplaza {id} con el ID de un producto existente, por ejemplo, 1.
- Descripción: Recupera los detalles de un producto específico por su ID.
- Paso en Postman:
    1. Crea una nueva solicitud con el método GET y la URL especificada.
    2. Prueba con un ID existente y verifica los detalles del producto en la respuesta.

<br/>

### 3. Crear un nuevo producto
- Método HTTP: POST
- URL: http://localhost:9081/productos
- Cuerpo (Body):
    - Selecciona el tipo JSON y utiliza el siguiente ejemplo:
    ```json
    {
    "nombre": "Producto A",
    "descripcion": "Descripción del Producto A",
    "precio": 100.0,
    "stock": 50
    }
    ```

- Descripción: Agrega un nuevo producto a la base de datos.
- Paso en Postman:
    1. Crea una nueva solicitud con el método POST, la URL especificada y el cuerpo en formato JSON.
    2. Ejecuta la solicitud y verifica que se cree un nuevo producto con el ID generado.

<br/>

### 4. Actualizar un producto existente
- Método HTTP: PUT
- URL: http://localhost:9081/productos/{id}
- Reemplaza {id} con el ID de un producto existente, por ejemplo, 1.
- Cuerpo (Body):
    - Selecciona el tipo JSON y utiliza el siguiente ejemplo:

    ```json
    {
    "nombre": "Producto B",
    "descripcion": "Descripción actualizada del produto B",
    "precio": 150.0,
    "stock": 40
    }
    ```

- Descripción: Actualiza los detalles de un producto existente.
- Paso en Postman:
    1. Crea una nueva solicitud con el método PUT, la URL especificada y el cuerpo en formato JSON.
    2. Ejecuta la solicitud y verifica que los cambios se reflejen al consultar el producto actualizado.

<br/>

### 5. Eliminar un producto por ID
- Método HTTP: DELETE
- URL: http://localhost:9081/productos/{id}
- Reemplaza {id} con el ID de un producto existente, por ejemplo, 1.
- Descripción: Elimina un producto específico por su ID.
- Paso en Postman:
    1. Crea una nueva solicitud con el método DELETE y la URL especificada.
    2. Ejecuta la solicitud y verifica que el producto ya no exista al intentar recuperarlo.

<br/>

### 6. Validación del microservcio ms-productos

- Prueba cada uno de los endpoints utilizando Postman.

- Documenta las respuestas obtenidas, incluyendo los códigos de estado HTTP (200, 201, 204, 404, etc.).

- Verifica que las operaciones cumplan con los objetivos definidos para la práctica.
 

### 7. Listar todos los deseos

- Método HTTP: GET
- URL: http://localhost:9084/deseos
- Descripción: Recupera una lista con todos los productos en la lista de deseos.
- Paso en Postman:
    1. Crea una nueva solicitud en la colección ms-deseos con el método GET y la URL especificada.
    2. Ejecuta la solicitud y verifica que se devuelva una lista en formato JSON.

<br/>

### 8. Agregar un producto a la lista de deseos

- Método HTTP: POST
- URL: http://localhost:9084/deseos/{idProducto}
- Reemplaza {idProducto} con el ID de un producto existente (por ejemplo, 1).
- Descripción: Agrega un producto a la lista de deseos usando su ID.
- Paso en Postman:
    1. Crea una nueva solicitud con el método POST y la URL especificada.
    2. Ejecuta la solicitud y verifica que se devuelva el producto agregado en la respuesta, incluyendo los detalles como idProducto, nombre, precio y fechaAgregado.

<br/>

### 9. Eliminar un producto de la lista de deseos

- Método HTTP: DELETE
- URL: http://localhost:9084/deseos/{idProducto}
- Reemplaza {idProducto} con el ID de un producto existente en la lista de deseos (por ejemplo, 1).
- Descripción: Elimina un producto específico de la lista de deseos utilizando su ID.
- Paso en Postman:
    1. Crea una nueva solicitud con el método DELETE y la URL especificada.
    2. Ejecuta la solicitud y verifica que el producto sea eliminado correctamente al listar los deseos nuevamente.

<br/>

### 10. Validación del microservcio ms-deseos

    - Prueba cada uno de los endpoints utilizando Postman.

    - Documenta las respuestas obtenidas, incluyendo los códigos de estado HTTP (200, 201, 204, 404, etc.).

    - Verifica que las operaciones cumplan con los objetivos definidos para la práctica.


### 11. Notas adicionales

-La lista de deseos no tiene persistencia; los datos se manejan en memoria. Por lo tanto, al reiniciar el microservicio, los datos se perderán.

- Asegúrate de que los IDs de productos utilizados para agregar o eliminar existan en el microservicio ms-productos (en el puerto 9081).


<br/>
<br/>

## Resultado Esperado