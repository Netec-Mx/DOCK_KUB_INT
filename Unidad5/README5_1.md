# Práctica 5.1 Spring Cloud Gateway

## Objetivo

- Al finalizar esta práctica, serás capaz de implementar y configurar un servicio de Spring Cloud Gateway en un clúster de Kubernetes. Esto incluye la creación y configuración del servicio, el enrutamiento de microservicios a través del Gateway utilizando `application.properties` y `application.yml`, la dockerización del Gateway, y el despliegue mediante Deployment y Service en Kubernetes, verificando su funcionalidad a través de Postman.
 

## Duración

180 minutos

<br/>

## Objetivo Visual

![Spring Cloud Gateway / Kubernetes](../images/u5_1_1.png)

<br/>
<br/>

## Instrucciones

### Paso 1. Crear el nuevo microservicio ms-gateway

#### 1. Crear el proyecto en Spring Tool Suite (STS)
- Crea un nuevo proyecto Maven llamado **ms-gateway**.
- Configura las opciones:
  - Lenguaje: Java 21.
  - Tipo de empaquetado: JAR.
  - Tipo: Maven

<br/>

#### 2. Agregar dependencia de Spring Cloud Gateway

- Incluye el **Spring Cloud Routing - Reactive Gateway** al crear el proyecto o agrégalo manualmente al archivo `pom.xml`.

<br/>

#### 3. Añadir inicializadores necesarios al archivo `pom.xml`
- Agrega las siguientes dependencias en el `pom.xml`:

```xml

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-kubernetes-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-kubernetes-client-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-kubernetes-client-loadbalancer</artifactId>
    </dependency>

```

- **Nota:** Verifica la compatibilidad de las versiones de Spring Cloud con la versión de Spring Boot de tu proyecto. Consulta [Spring Cloud Compatibility Matrix](https://spring.io/projects/spring-cloud#overview).

<br/>

#### 4. Habilitar cliente de descubrimiento
- En la clase principal `MsGatewayApplication`, añade la anotación `@EnableDiscoveryClient`:

```java

// Líneas package e imports omitidos

import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class MsGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(MsGatewayApplication.class, args);
    }
}
```
<br/>

#### 5. Configurar el archivo `application.properties`

- Agrega las siguientes propiedades básicas:

```properties
spring.application.name=ms-gateway
server.port=9099
```

<br/>

#### 6. Configurar rutas con `application.yml`

- Crea un archivo `application.yml` para definir las rutas necesarias:
  
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: ms-productos
          uri: lb://ms-productos
          predicates:
            - Path=/api1/**
          filters:
            - StripPrefix=1
        - id: ms-deseos
          uri: lb://ms-deseos
          predicates:
            - Path=/api2/**
          filters:
            - StripPrefix=1
```

<br/>

#### 7. Decidir el formato de configuración
- Decide si prefieres utilizar únicamente `application.properties` o `application.yml` para todas las configuraciones del proyecto. Elimina el archivo que no se usará para evitar conflictos.

<br/>

#### 8. Crear el artefacto JAR
- Genera el artefacto JAR del microservicio ejecutando el siguiente comando en la raíz del proyecto:

```bash
.\mvnw clean package
```

<br/>

#### 9. Crear el Dockerfile

- Escribe un archivo `Dockerfile` para dockerizar el microservicio:

```dockerfile
# 1. Imagen base
FROM openjdk:21-jdk-slim

# 2. Establecer el directorio de trabajo
WORKDIR /app

# 3. Copiar el archivo JAR generado
COPY target/ms-gateway-0.0.1-SNAPSHOT.jar app.jar

# 4. Exponer el puerto utilizado por el microservicio
EXPOSE 9099

# 5. Comando de inicio
ENTRYPOINT ["java", "-jar", "app.jar"]
```

<br/>

#### 10. Construir, etiquetar y registrar la imagen Docker

- Construye la imagen Docker:

```bash
docker build -t ms-gateway:<tu_version> .
```

- Etiqueta y sube la imagen a Docker Hub:

```bash
docker login
docker tag ms-gateway:<version> <tu_usuario_dockerhub>/ms-gateway:<tu_version>
docker push <tu_usuario_dockerhub>/ms-gateway:<tu_version>
```

#### 11. Verificar el registro de la imagen
- Asegúrate de que la imagen se encuentra registrada correctamente en Docker Hub ejecutando:

```bash
docker pull <tu_usuario_dockerhub>/ms-gateway:<tu_version>
``` 

Con estos pasos finalizados, tendrás tu microservicio **ms-gateway** listo para desplegar en Kubernetes.

<br/>
<br/>

### Paso 2. Desplegar el microservicio ms-gateway en el clúster de Kubernetes

#### 1. Crear el manifiesto YAML para el despliegue del microservicio
- Escribe un manifiesto YAML que incluya:
  - Un **Deployment** que utilice la imagen Docker creada en el Paso 1 como base para el contenedor.
  - Configura un número adecuado de réplicas según las necesidades del servicio.
  - Define las siguientes características opcionales (recomendadas):
    - **Resources**: límites y solicitudes de CPU y memoria para optimizar el uso de recursos del clúster.
    - **Probes**: añade `livenessProbe` y `readinessProbe` para garantizar la disponibilidad del servicio y evitar tráfico hacia Pods no saludables.
  - Asegúrate de incluir los puertos necesarios para el contenedor.

<br/>

#### 2. Crear el manifiesto YAML para el Service

- Define un **Service** en Kubernetes para exponer el microservicio **ms-gateway**:
  - Tipo de servicio: `LoadBalancer` (para exponer el servicio externamente y permitir el acceso desde fuera del clúster).
  - Configura el puerto externo y el puerto del contenedor de acuerdo con la configuración del microservicio (e.g., puerto 9099).
  - Incluye las etiquetas necesarias para asociar el Service con los Pods del Deployment.

<br/>

#### 3. Aplicar los manifiestos YAML al clúster

- Aplica los manifiestos al clúster con el siguiente comando:

```bash
kubectl apply -f <nombre-del-archivo-deployment>.yaml
kubectl apply -f <nombre-del-archivo-service>.yaml
```

<br/>

#### 4. Inspeccionar los componentes de Kubernetes
- Verifica que los Pods, Deployment y Service se hayan creado correctamente utilizando los comandos:

```bash
kubectl get pods
kubectl get deployment ms-gateway
kubectl get service ms-gateway
```

- Asegúrate de que los Pods estén en estado `Running` y que el Service tenga una dirección IP asignada (si usas `LoadBalancer`).

<br/>
<br/>

### Paso 3. Consumo del nuevo microservicio **ms-gateway**

#### 1. Verificar la dirección IP y el puerto del servicio

- Identifica la dirección IP y el puerto expuesto por el **Service** de Kubernetes para **ms-gateway**. 

  - Si usaste un `LoadBalancer`, utiliza el siguiente comando para obtener la dirección IP externa:

    ```bash
    kubectl get services -o wide
    ```

- Anota la dirección IP y el puerto asignados.

<br/>

#### 2. Confirmar la disponibilidad de los microservicios dependientes

- Asegúrate de que los microservicios **ms-productos** y **ms-deseos** estén en un estado saludable y listos para ser consumidos por **ms-gateway**.

  - Ejecuta el siguiente comando para verificar su estado:

    ```bash
    kubectl get pods
    ```

- Realiza una prueba básica para cada microservicio directamente, usando **Postman** o `curl`, para confirmar que están respondiendo correctamente.

<br/>

#### 3. Probar la conectividad hacia **ms-productos** a través de **ms-gateway**

- Utiliza **Postman** o `curl` para consumir las rutas configuradas en **ms-gateway** que se redirigen a **ms-productos**. Por ejemplo:

    ```bash
    curl http://<IP_DEL_GATEWAY>:<PUERTO>/api1/productos
    ```

- Asegúrate de que los datos de **ms-productos** se obtienen correctamente a través de **ms-gateway**.

<br/>

#### 4. Agregar un nuevo producto a través de **ms-gateway**

- Envía una solicitud POST con los datos de un nuevo producto utilizando la ruta de **ms-gateway** que redirige a **ms-productos**. Ejemplo con `curl`:

    ```bash
  curl -X POST http://<IP_DEL_GATEWAY>:<PUERTO>/productos \
     -H "Content-Type: application/json" \
     -d '{
          "id": 2,
          "nombre": "Producto B",
          "descripcion": "Descripción del Producto B",
          "precio": 200.5,
          "stock": 3
     }'

    ```

- Confirma que la solicitud fue aceptada y que el producto fue creado correctamente.

<br/>

#### 5. Verificar la creación del producto

- Usa la ruta de consulta configurada en **ms-gateway** para verificar que el nuevo producto aparece en la lista de productos de **ms-productos**:

    ```bash
    curl http://<IP_DEL_GATEWAY>:<PUERTO>/api/productos
    ```

- Valida que el producto agregado esté presente en la respuesta.


- Usa la ruta de consulta configurada en **ms-gateway** para verificar que el nuevo producto aparece en la lista de productos de **ms-productos**:

    ```bash
    curl http://<IP_DEL_GATEWAY>:<PUERTO>/api/productos
    ```

- Usa la ruta de consulta configurada en **ms-gateway** para verificar que el puedes consumir el nuevo producto desde **ms-deseos**:

    ```bash
    curl -s -X POST http://<IP_DEL_GATEWAY>:<PUERTO>/api/deseos/2
    ```
<br/>

Con estos pasos, habrás comprobado la funcionalidad del **ms-gateway**, la interacción con los microservicios dependientes, y la correcta propagación de solicitudes entre los servicios.

<br/>
<br/>

## Resultados Esperados


