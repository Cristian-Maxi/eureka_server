# 🧩 Proyecto de Microservicios con Spring Boot

Este proyecto es una implementación de una arquitectura de microservicios basada en Spring Boot. Utiliza tecnologías modernas como Redis, RabbitMQ y PostgreSQL para resolver problemas empresariales comunes de manera escalable, eficiente y modular.

## 🔧 Tecnologías Utilizadas

- Java 21
- Spring Boot 3.4.4
- Maven
- PostgreSQL
- Redis
- RabbitMQ
- Eureka Server (Service Discovery)
- Spring Cloud Gateway
- MapStruct
- Spring Data JPA
- Spring Validation
- Java Mail Sender
- ITextPDF
- Mailtrap
- Docker / Podman
- Postman / DBBeaver
- Open Api (Swagger)
- Junit / Mockito
- H2 Database (Testing)
- Jacoco + informe en HTML
- SonarQube (Revisión de cobertura)
- Zipkin (Para Trazabilidad)

## 📌 Eureka Server

Vamos a utilizar Eureka Server para localizar y registrar servicios con los que se desea interactuar, además nos va a ayudar con el balanceo de carga y la tolerancia a fallos.

---

### 🔸 Pre-requisitos

- Tener instalado `Podman` y `Podman Compose`.
- Crear un archivo `.env` con las siguientes variables:
```.env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=myappdb

SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=postgres

SECRET_KEY=B374A26A71421437AA024E4FADD5B478FDFF1A8EA6FF12F6FB65AF2720B59CCF
```

- docker-compose.yml:

```
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: springboot_postgres
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - spring-network

  redis:
    image: redis:7.2
    container_name: redis_cache
    restart: always
    ports:
      - "6379:6379"
    networks:
      - spring-network

  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    restart: always
    ports:
      - "5672:5672"   # Puerto de conexión para microservicios
      - "15672:15672" # Puerto de panel web de administración
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    networks:
      - spring-network

  eureka:
    build: ./eureka_server
    container_name: eureka_server
    ports:
      - "8761:8761"
    networks:
      - spring-network

  gateway:
    build: ./cloud-gateway
    container_name: gateway_service
    ports:
      - "8080:8080"
    depends_on:
      - eureka
    environment:
      - SECRET_KEY=${SECRET_KEY}
    networks:
      - spring-network

  pointsalecost:
    build: ./Point_of_Sale_Cost-Microservice
    container_name: pointsalecost_service
    depends_on:
      - postgres
      - redis
      - eureka
    environment:
      - SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}
    networks:
      - spring-network

  accreditations:
    build: ./Accreditation-Microservice
    container_name: accreditations_service
    depends_on:
      - postgres
      - redis
      - eureka
      - rabbitmq
    environment:
      - SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}
    networks:
      - spring-network

  usermicroservice:
    build: ./Users-Microservice
    container_name: usermicroservice_service
    depends_on:
      - postgres
      - redis
      - eureka
      - rabbitmq
    environment:
      - SPRING_DATASOURCE_USERNAME=${SPRING_DATASOURCE_USERNAME}
      - SPRING_DATASOURCE_PASSWORD=${SPRING_DATASOURCE_PASSWORD}
      - SECRET_KEY=${SECRET_KEY}
    networks:
      - spring-network

  emailrabbitmq:
    build: ./emailRabbitMQ-Microservice
    container_name: email_rabbitmq_service
    depends_on:
      - rabbitmq
    environment:
      - SECRET_KEY=${SECRET_KEY}
    ports:
      - "8084:8084"
    networks:
      - spring-network

volumes:
  postgres_data:

networks:
  spring-network:
    driver: bridge


```

- Comando para construir y levantar con Podman Compose:
`podman compose up --build`

## ✅ Estructura de la Carpeta

![Estructuras de carpetas](https://github.com/user-attachments/assets/4f43aa01-d1de-4aee-a90b-e7b670e94f07)


## 💻​ Diagrama del Flujo General de la Aplicación

![Diagrama del Flujo General](https://github.com/user-attachments/assets/c9106c67-a151-4f4c-9fb2-b2c6f1630faa)
