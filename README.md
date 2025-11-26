# Proyecto Final · E-commerce con Spring Boot y Kafka

## Curso
- Curso Spring Boot & Kafka

## Nombre 
- Judith Sayda Quelca Nina  
- CI 4373568

## REPOSITORIOS
- ecommerce-product-service (9495)

https://github.com/judithquelca/ecommerce-product-service

- ecommerce-ecommerce-order-service (8081)

https://github.com/judithquelca/ecommerce-order-service

- ecommerce-inventory-service (8082)

https://github.com/judithquelca/ecommerce-inventory-service


### Arquitectura y estructura

#### Capas de cada servicio

- product-service

 ![Imagen de contenedor descargada](recursos/productCapas.png)

- order-service

 ![Imagen de contenedor descargada](recursos/orderCapas.png)

- inventory-service

 ![Imagen de contenedor descargada](recursos/inventoryCapas.png)


#### Archivos de configuración
- Los archivos de configuración que se tiene son los siguientes:

  - application.yml
  - application-dev.yml
  - application-prod.yml
  - ValidationMessages.properties

![Imagen de contenedor descargada](recursos/archivosConfiguracion.png)


### Funcionalidad REST y validaciones

- **product-service**

  - creación de categoria
  ![Imagen de contenedor descargada](recursos/createCategory.png)
  
   - creación de producto
  ![Imagen de contenedor descargada](recursos/createProducts.png)
  
   - lista productos
  ![Imagen de contenedor descargada](recursos/getAllProducts.png)
  
   - búsqueda producto por id
  ![Imagen de contenedor descargada](recursos/getProductById.png)

- **order-service**

	- creación de orden de producto
	![Imagen de contenedor descargada](recursos/createOrderLaptop10.png)
	
	- lista ordenes
	![Imagen de contenedor descargada](recursos/listOrder.png)	
	
	- búsqueda orden  por id
	![Imagen de contenedor descargada](recursos/orderId.png)	
	
	- estado PENDING/CONFIRMED/CANCELLED
	![Imagen de contenedor descargada](recursos/orderPendiente.png)
	
	![Imagen de contenedor descargada](recursos/orderConfirmed.png)	
	
	![Imagen de contenedor descargada](recursos/orderCancelada.png)

- **inventory-service**

 - creación de inventario
 ![Imagen de contenedor descargada](recursos/createInventory.png)

 - lista todos los items inventarios
  ![Imagen de contenedor descargada](recursos/getAllInventory.png)
  
 - búsqueda inventario por producto 
  ![Imagen de contenedor descargada](recursos/getInventoryByPoructId.png)
  
 - búsqueda inventario por id
  ![Imagen de contenedor descargada](recursos/getInventoryById.png) 
 
 - verifica la actualización de inventario 
  ![Imagen de contenedor descargada](recursos/VerifyInventoriLaptop10.png ) 


- **Validaciones**
  - Para las validaciones se realizo lo siguiente:

    - Crear el archivo de ValidationMessages.properties
  
    - Adicionar la dependencia en archivo pom.xml

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>

    - Actualizar el archivo application.yml

		spring:
		  messages:
			basename: ValidationMessages

    - Parametrizar las validaciones para order-service

		order.product.notblank=El ID del producto es requerido
		order.product.positive=El ID del producto debe ser positivo
		order.quantity.notblank =La cantidad es requerida
		order.quantity.positivew=La cantidad debe ser positiva
		order.quantity.max=La cantidad no puede exceder 100
		order.name.notblank=El nombre del cliente es requerido
		order.name.min=El nombre debe tener entre 3 y 100 caracteres
		order.email.notblank=El email del cliente es requerido
		order.email.email=El email debe ser válido
		order.totalAmount.notblank=El monto total es requerido
		order.totalAmount.positive=El monto total debe ser positivo


		public record OrderRequest(

				@NotNull(message = "{order.product.notblank}")
				@Positive(message = "{order.product.positive}")
				Long productId,

				@NotNull(message = "{order.quantity.notblank}")
				@Positive(message = "{order.quantity.positive}")
				@Max(value = 100, message = "{order.quantity.max}")
				Integer quantity,

				@NotBlank(message = "{order.name.notblank}")
				@Size(min = 3, max = 100, message = "{order.name.min}")
				String customerName,

				@NotBlank(message = "{order.email.notblank}")
				@Email(message = "{order.email.email}")
				String customerEmail,

				@NotNull(message = "{order.totalAmount.notblank}")
				@Positive(message = "{order.totalAmount.positive}")
				BigDecimal totalAmount

		) {
		}

	![Imagen de contenedor descargada](recursos/orderValid.png)
	
	![Imagen de contenedor descargada](recursos/validPositive.png)

	- Parametrizar las validaciones para inventoryservice
	
		inventory.product.notblank=El ID del producto es requerido
		inventory.product.name=El nombre del producto es requerido
		inventory.stock.notblank =El stock inicial es requerido
		inventory.stock.positive=El stock inicial debe ser no negativo
		

		public record InventoryItemRequest (

			@NotNull(message = "{inventory.product.notblank}")
			Long productId,

			@NotBlank(message = "{inventory.product.name}")
			String productName,

			@NotNull(message = "{inventory.stock.notblank}")
			@Min(value = 0, message = "{inventory.stock.positive}")
			Integer initialStock
		){
		}

 - **Adición de la clase GlobalExceptionHandler**

	- Se creo las siguientes clases dentro del paquete exception
		- ErrorResponse.java
        - ResourceNotFoundException.java
		- GlobalExceptionHandler.java
		
    - Se actualizó la clase OrderService.java para lanzar excepciones
        
			  @Transactional(readOnly = true)
				public OrderResponse findById(Long id) {
					Order order = orderRepository.findById(id)
							.orElseThrow(() -> new OrderNotFoundException("Orden " + id + " no encontrado"));
					return OrderMapper.toResponse(order);
				}

		![Imagen de contenedor descargada](recursos/orderException.png)

		![Imagen de contenedor descargada](recursos/inventoryException.png)
		
		![Imagen de contenedor descargada](recursos/validaciones.png)
		
		![Imagen de contenedor descargada](recursos/ordenNoEncontrada.png)


### Kafka

- Actualización de Topics (5 particiones, 1 réplica):
   - ecommerce.products.created
   - ecommerce.orders.placed
   - ecommerce.orders.confirmed
   - ecommerce.orders.cancelled
  
 - Verificación que Kafka este corriendo

	- docker compose ps
	- docker exec -it kafka bash

 - Actualmente esta es la lista topics existentes
     
	 - kafka-topics --bootstrap-server localhost:9092 --list
	 
    ![Imagen de contenedor descargada](recursos/listKafkaActual.png)

			
	- Con los siguientes comandos se actualiza el número de Topics
  
		  kafka-topics --bootstrap-server localhost:9092 --alter --topic ecommerce.products.created --partitions 5
		  kafka-topics --bootstrap-server localhost:9092 --alter --topic ecommerce.orders.placed --partitions 5
		  kafka-topics --bootstrap-server localhost:9092 --alter --topic ecommerce.orders.confirmed --partitions 5
		  kafka-topics --bootstrap-server localhost:9092 --alter --topic ecommerce.orders.cancelled --partitions 5
		  kafka-topics --bootstrap-server localhost:9092 --alter --topic ecommerce.inventory.updated --partitions 5

		![Imagen de contenedor descargada](recursos/updateParticionKafka.png)
		
		![Imagen de contenedor descargada](recursos/productsTopic.png)
		
		![Imagen de contenedor descargada](recursos/orderPlacedTopic.png)
		
		![Imagen de contenedor descargada](recursos/orderConfirmTopic.png)
		
		![Imagen de contenedor descargada](recursos/orderCancelledTopic.png)
		
		![Imagen de contenedor descargada](recursos/inventoryTopic.png)

  - Configuración de kafka con `spring.kafka` y `spring.json.type.mapping`.
	
	- productservice
		
		![Imagen de contenedor descargada](recursos/inventoryTopic.png)
		  
		![Imagen de contenedor descargada](recursos/productKafka.png)  
  
	- orderservice
		
		![Imagen de contenedor descargada](recursos/orderKafka.png)
	  
	- inventoryservice
		
		![Imagen de contenedor descargada](recursos/inventoryKafka.png)
  
  
-  Flujo con lo que se obtuvo de postman  

	![Imagen de contenedor descargada](recursos/tresProducts.png)
		
	![Imagen de contenedor descargada](recursos/ordenesCreadas.png)
		
	![Imagen de contenedor descargada](recursos/ordenConfirmada.png)
		
	![Imagen de contenedor descargada](recursos/ordenCancelada.png)
	
	![Imagen de contenedor descargada](recursos/kafkaCanceled.png)	
		

### Bases de datos y modelos

- Tres bases PostgreSQL (`ecommerce`, `ecommerce_orders`, `ecommerce_inventory`)

 - Se crea la BD con los siguientes script (desde la consola de postgres)
 
		CREATE DATABASE ecommerce;
		CREATE DATABASE ecommerce_orders;
		CREATE DATABASE ecommerce_inventory;

	![Imagen de contenedor descargada](recursos/baseDatos.png)

 - En el archivo doker-compose.yml se debe configurar lo siguiente: 
 
		  postgres:
			image: postgres:15-alpine
			container_name: product-db
			restart: unless-stopped
			environment:
			  POSTGRES_DB: ecommerce
			  POSTGRES_USER: ecommerce_user
			  POSTGRES_PASSWORD: ecommerce_password
			ports:
			  - "5432:5432"
			volumes:
			  - postgres-data:/var/lib/postgresql/data

	![Imagen de contenedor descargada](recursos/dockerPostgres.png)
	
 - Se adiciona en el archivo pom.xml, la siguiente dependencia
 
		<dependency>
			<groupId>org.postgresql</groupId>
				<artifactId>postgresql</artifactId>
				<scope>runtime</scope>
		</dependency>

	![Imagen de contenedor descargada](recursos/pomPostgres.png)

- Entidades con relaciones Product–Category 1:N
	
	![Imagen de contenedor descargada](recursos/claseProductCategory.png)
	
	
- Diagrama con los tres microservicios con kafka
	
	![Imagen de contenedor descargada](recursos/tresMicroservicios.png)	
	

### Configuración y variables 

- En el archivo `application.yml` se tiene las variables para ser parametrizadas como variables de entorno

![Imagen de contenedor descargada](recursos/ymlConfigura.png)

- Configuración de variables de entorno

![Imagen de contenedor descargada](recursos/variablesEntorno.png)

![Imagen de contenedor descargada](recursos/varEntornoOrder.png)


- Entorno de desarrollo

![Imagen de contenedor descargada](recursos/entornoDev.png)

- Se activa desde la siguiente opción: profiles.active

![Imagen de contenedor descargada](recursos/entornoProductivo.png)

- Al momento de ejecutar la aplicación se puede verificar en los logs

![Imagen de contenedor descargada](recursos/verificaProd.png)


