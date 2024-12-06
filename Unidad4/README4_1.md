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

1. **Agrega las dependencias necesarias en el archivo `pom.xml`**:
   - Spring Boot Starter Actuator.
   - Spring Cloud Kubernetes Config.
   - Spring Cloud Kubernetes Discovery.
   
   **Nota:** Usa las versiones compatibles con Spring Boot y Kubernetes.

<br/>

2. **Configura los archivos `application.properties` o `application.yml`**:
   - Define los valores para:
     - `spring.application.name` con el nombre del microservicio.
     - Puerto en el que correrá el microservicio.
   - Agrega las propiedades necesarias para habilitar la integración con Kubernetes ConfigMaps y los endpoints de Actuator.

<br/>

3. **Anota la clase principal de cada microservicio con `@EnableDiscoveryClient`**:
   - Abre el archivo de la clase principal de la aplicación, generalmente llamado `MsProductosApplication` o `MsDeseosApplication`.
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

   **Nota:** Repite esta configuración para ambos microservicios (`ms-productos` y `ms-deseos`). 

<br/>

4. **Verifica que los endpoints de Actuator estén habilitados**:
   - Configura los endpoints necesarios en `application.properties` o `application.yml`:
     ```properties
     management.endpoints.web.exposure.include=*
     management.endpoint.health.show-details=always

     # Configuracion para Kubernetes DNS
     spring.cloud.kubernetes.discovery.enabled=true
     spring.cloud.kubernetes.secrets.enable-api=true
     spring.cloud.kubernetes.discovery.all-namespaces=true
     
     ```
   - Esto permite que Kubernetes pueda consultar los estados de `liveness` y `readiness`.

<br/>

5. **Actualizar el cliente Feign en el microservicio `ms-deseos`:**
   - Modifica la clase `ProductoClient` para usar el servicio de descubrimiento en lugar de una URL estática. 

   - Elimina la referencia a la propiedad `ms-peliculas.url` en la anotación `@FeignClient`.

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
		public Long getId() {
			return id;
		}

		public void setId(Long id) {
			this.id = id;
		}

		public String getNombre() {
			return nombre;
		}

		public void setNombre(String nombre) {
			this.nombre = nombre;
		}

		public String getDescripcion() {
			return descripcion;
		}

		public void setDescripcion(String descripcion) {
			this.descripcion = descripcion;
		}

		public Double getPrecio() {
			return precio;
		}

		public void setPrecio(Double precio) {
			this.precio = precio;
		}

		public Integer getStock() {
			return stock;
		}

		public void setStock(Integer stock) {
			this.stock = stock;
		}
	}
}

   ```
 
<br/>

6. **Crear los artefactos para cada microservicio**

    1. Asegúrate de que el código fuente de cada microservicio (`ms-productos` y `ms-deseos`) esté actualizado y sin errores en tu entorno de desarrollo.

    2. Usa tu herramienta de construcción (Maven) para compilar el proyecto y generar los artefactos JAR correspondientes.

    3. Verifica que los archivos JAR generados estén en la carpeta `target` de cada microservicio. Los artefactos deben tener nombres como `ms-productos-<versión>.jar` y `ms-deseos-<versión>.jar`.

    4. Asegúrate de que los artefactos cumplen con los requisitos funcionales y de configuración antes de continuar con los siguientes pasos del despliegue.

    **Nota:** Estos artefactos serán usados en la construcción de imágenes Docker para cada microservicio.

<br/>

7. **Registra las nuevas imagenes en Docker Hub**

    1. Inicia sesión en Docker Hub desde la terminal para autenticarte con tu cuenta.
   
    2. Etiqueta las imágenes locales para asociarlas a tu repositorio en Docker Hub.

    3. Sube las imágenes etiquetadas a tu repositorio en Docker Hub.

    4. Verifica en la plataforma de Docker Hub que las imágenes `ms-productos` y `ms-deseos` se hayan registrado correctamente en tu cuenta.

    **Nota:** Recuerda utilizar tu nombre de usuario de Docker Hub al etiquetar las imágenes.

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

