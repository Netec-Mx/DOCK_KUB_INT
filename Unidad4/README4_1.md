# Práctica 4.1 Spring Cloud Kubernetes

## Objetivo

- Al finalizar esta práctica, serás capaz de integrar y aplicar todos los conocimientos adquiridos en la unidad 4 para configurar, desplegar y probar microservicios Spring Boot en un clúster de Kubernetes, utilizando Spring Cloud Kubernetes, ConfigMaps, configuración de entornos, probes de liveness y readiness, y ajustes de recursos para contenedores, validando la implementación mediante herramientas como Postman y observando el comportamiento del LoadBalancer y metadata de los Pods.

<br/>

## Objetivo Visual

![Microservicios Caso Estudio](../images/u4_1_1.png)

<br/>

## Duración

180 minutos

<br/>
<br/>


## Instrucciones

### **Paso 1: Configuración de Spring Cloud Kubernetes en los Microservicios**

1. **Agrega las dependencias necesarias en el archivo `pom.xml`** para cada microservicio:

   - Spring Boot Starter Actuator.
   - Spring Cloud Kubernetes Config.
   - Spring Cloud Kubernetes Discovery.
   - Spring Cloud Kubernetes Client
   - Spring Cloud Kubernetes Client Config
   - Spring Cloud Kubernetes Client LoadBalancer
   

   
   **Notas:** 
   1. Usa las versiones compatibles con Spring Boot y Kubernetes [Versiones Soportadas](https://github.com/spring-cloud/spring-cloud-release/wiki/Supported-Versions).

    2. En la elaboración del material de curso (Nov/2024) se uso la versión 3.1.2 con Spring Boot 3.3.5

<br/>

2. **Anota la clase principal de cada microservicio con `@EnableDiscoveryClient`**:

Anota la clase principal de cada aplicación con @EnableDiscoveryClient para habilitar la capacidad de descubrimiento en Kubernetes.

   - Abre el archivo de la clase principal de la aplicación, generalmente llamado `MsProductosApplication` y `MsDeseosApplication`.

   - Agrega la anotación `@EnableDiscoveryClient` para permitir que los microservicios se registren y descubran en Kubernetes.

   ```java
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;

   @SpringBootApplication
   @EnableDiscoveryClient
   public class MsProductosApplication {
       public static void main(String[] args) {
           SpringApplication.run(MsProductosApplication.class, args);
       }
   }
   ```

   **Archivo**: `MsDeseosApplication.java`

   ```java
    package com.example.msdeseos;

    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    @EnableDiscoveryClient
    public class MsDeseosApplication {
        public static void main(String[] args) {
            SpringApplication.run(MsDeseosApplication.class, args);
        }
    }
   ```

<br/>


3. **Actualizar el cliente Feign en el microservicio `ms-deseos`:**
 
 Modifica el cliente Feign ProductoFeignClient para usar el servicio de descubrimiento en lugar de una URL estática

   - Modifica la clase `ProductoFeignClient` para usar el servicio de descubrimiento en lugar de una URL estática. 

   - Elimina la referencia el atributo url en la anotación `@FeignClient`.

**Archivo: `ProductoFeignClient.java`**

```java
    package com.netec.app.feign;

    import org.springframework.cloud.openfeign.FeignClient;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PathVariable;

    @FeignClient(name = "ms-productos")
    public interface ProductoFeignClient {

        @GetMapping("/productos/{id}")
        Producto obtenerProductoPorId(@PathVariable Long id);

        class Producto {
            private Long id;
            private String nombre;
            private String descripcion;
            private Double precio;
            private Integer stock;

            // Getters y Setters
           
        }
    }

```
 
<br/>

4. **Modifcar el controlador para incluir metadatos del Pod** 

- Modifica el controlador de tu microservicio para incluir en la respuesta el nombre y la dirección IP del Pod que procesa la solicitud.

- **Archivo**: `ProductoController.java`

```java
 
 // Package & Imports omitidos

@RestController
@RequestMapping("/productos")
public class ProductoController {

	private final IProductoService productoService;

    @Autowired
    private Environment environment;

    public ProductoController(ProductoServiceImpl productoService) {
        this.productoService = peliculaService;
    }

    @GetMapping
    public Map<String, Object> listarTodos() {
        return Map.of(
            "POD_NAME", environment.getProperty("POD_NAME", "Unknown"),   
            "POD_ID", environment.getProperty("POD_ID", "Unkown"), 
            "productos", pproductoService.listarTodos());
    }
    // Líneas omitidas
}

```

5. **Configura los archivos `application.properties` o `application.yml`**:

- Agrega las propiedades necesarias para habilitar la integración con Kubernetes ConfigMaps y los endpoints de Actuator.

- **Nota**: Algunas propiedades ya se configuraron en las prácticas de la Unidad 3, solo verifica que se encuentren y agrega las necesarias para Spring Cloud Kubernetes


```properties

# Spring Cloud Kubernetes
spring.cloud.kubernetes.disovery.enabled=true
spring.cloud.kubernetes.secrets.enable-api=true
spring.cloud.kubernetes.discovery.all-namespaces=true

# Spring Boot Actuator
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.endpoint.health.probes.enabled=true
management.health.livenessstate.enabled=true
management.health.readinessstate.enabled=true

# Perfiles
spring.profiles.active=dev

```
<br/>



6. **Crear los artefactos para cada microservicio**

- Asegúrate de que el código fuente de cada microservicio (`ms-productos` y `ms-deseos`) esté actualizado y sin errores en tu entorno de desarrollo.

- Usa tu herramienta de construcción (Maven) para compilar el proyecto y generar los artefactos JAR correspondientes.

- Verifica que los archivos JAR generados estén en la carpeta `target` de cada microservicio. Los artefactos deben tener nombres como `ms-productos-<versión>.jar` y `ms-deseos-<versión>.jar`.

- Asegúrate de que los artefactos cumplen con los requisitos funcionales y de configuración antes de continuar con los siguientes pasos del despliegue.

- **Nota:** Estos artefactos serán usados en la construcción de imágenes Docker para cada microservicio.

<br/>

7. **Registra las nuevas imagenes en Docker Hub**

- Inicia sesión en Docker Hub desde la terminal para autenticarte con tu cuenta.
   
- Etiqueta las imágenes locales para asociarlas a tu repositorio en Docker Hub.

- Sube las imágenes etiquetadas a tu repositorio en Docker Hub.

- Verifica en la plataforma de Docker Hub que las imágenes `ms-productos` y `ms-deseos` se hayan registrado correctamente en tu cuenta.

- **Nota:** Recuerda utilizar tu nombre de usuario de Docker Hub al etiquetar las imágenes.

<br/>


### **Paso 2: Crear ConfigMaps para las Configuraciones de los Microservicios**

1. **Codifica un archivo YAML para ConfigMap**:
   
- Define un ConfigMap para cada microservicio (`ms-productos` y `ms-deseos`) que incluya configuraciones personalizadas, como propiedades específicas del entorno.

**Elementos a incluir en el ConfigMap:**
    - Nombre del ConfigMap.
    - Clave-valor de las propiedades (`app.name`, `app.environment`).
   
**Comando para crear el ConfigMap:**

   ```bash
   kubectl apply -f ms-productos-configmap.yml
   kubectl apply -f ms-deseos-configmap.yml
   ```
   
   **Salida esperada:**
   ```
   configmap/ms-productos-config creado
   configmap/ms-deseos-config creado
   ```


<br/>


### **Paso 3: Configurar Liveness y Readiness Probes**

1. **Codifica un YAML para los Pods de los microservicios**:
   - Define las `probes` en la especificación del contenedor:
     - **Liveness Probe**: Utiliza el endpoint `/actuator/health/liveness`.
     - **Readiness Probe**: Utiliza el endpoint `/actuator/health/readiness`.
   
   - Configura el tiempo de inicio, período de sondeo y tiempo de espera.

   **Comando para aplicar el archivo de despliegue:**
   ```bash
   kubectl apply -f ms-productos-deployment.yml
   kubectl apply -f ms-deseos-deployment.yml
   ```


   **Salida esperada:**
   ```
   deployment.apps/ms-productos creado
   deployment.apps/ms-deseos creado
   ```

<br/>

### **Paso 4: Ajustar Recursos para los Contenedores**

1. **Configura los recursos del contenedor en los YAML de despliegue**:
   
   - **Request**:
     - CPU: `100m`.
     - Memoria: `256Mi`.
   - **Limit**:
     - CPU: `500m`.
     - Memoria: `512Mi`.

2. **Verifica los límites aplicados:**

   ```bash
   kubectl describe pod <nombre-del-pod>
   ```

<br/>


### **Paso 5: Configurar y Probar el Balanceador de Carga**

1. **Codifica el YAML para el servicio de tipo LoadBalancer**:
   
   - Define los puertos y el selector correspondiente.
   
   - Incluye el tipo de servicio como `LoadBalancer`.

   **Comando para crear el servicio:**
   ```bash
   kubectl apply -f ms-productos-service.yml
   kubectl apply -f ms-deseos-service.yml
   ```

   **Salida esperada:**
   ```
   service/ms-productos creado
   service/ms-deseos creado
   ```

2. **Verifica la dirección IP asignada:**
   ```bash
   kubectl get services
   ```
   
   **Salida esperada:**
   ```
   NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
   ms-productos     LoadBalancer   10.96.123.45     35.227.145.1  8080:31234/TCP   2m
   ms-deseos        LoadBalancer   10.96.123.46     35.227.145.2  8081:31235/TCP   2m
   ```

<br/>

### **Paso 6: Validación Final con Postman**

1. **Realiza las pruebas en Postman**:

   - Usa las direcciones IP y puertos expuestos para probar los endpoints `/actuator/health`, `/api/productos`, y `/api/deseos`.

2. **Valida las respuestas y el comportamiento del balanceador**:
   - Observa los cambios en las respuestas para diferentes Pods (metadata de Pods, nombres).

<br/>

### **Paso 7: Observación de Pods y Logs**

1. **Monitorea los Pods en ejecución**:
   ```bash
   kubectl get pods -o wide
   ```
   
2. **Revisa los logs de los microservicios**:
   ```bash
   kubectl logs <nombre-del-pod>
   ```

   **Salida esperada:**
   ```
   2024-12-05 12:34:56 INFO  com.example.Application - Started Application in 5.23 seconds
   ```


<br/>
<br/>

## **Resultados Esperados**

1. Visualización de los ConfigMaps y Secrets configurados para los microservicios en el clúster de Kubernetes.

2. Visualización clara y organizada de los Deployments y Pods, confirmando que están desplegados correctamente.

3. Visualización de los Services creados, verificando sus tipos, puertos y conectividad.

4. Validación del balanceo de carga al enviar solicitudes al servicio de productos, distribuyendo las peticiones entre las réplicas configuradas.

5. Acceso y validación de los endpoints de Actuator en cualquiera de los microservicios, confirmando el correcto funcionamiento de los probes y métricas.

6. Ejecución y respuesta adecuada al consumir los endpoints estándar de Actuator (`/health`, `/metrics`, etc.).

7. Consumo exitoso de los endpoints principales ("normales") de los microservicios, confirmando que responden de manera correcta y esperada.