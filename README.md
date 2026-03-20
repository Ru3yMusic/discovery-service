# discovery-service

Registro de servicios (**Eureka Server**) para toda la arquitectura de **RUBY MUSIC**. Es el primer servicio que debe arrancar: todos los microservicios se registran aquí y lo consultan para resolver ubicaciones entre sí.

---

## Responsabilidad

Actúa como directorio central donde cada microservicio:
1. **Se registra** al arrancar (publicando su host y puerto)
2. **Consulta** para descubrir la ubicación de otros servicios (load-balanced via `lb://`)

Sin este servicio corriendo, ningún otro microservicio puede resolver dependencias ni el api-gateway puede enrutar tráfico.

---

## Stack

| Componente | Versión |
|---|---|
| Java | 21 |
| Spring Boot | 3.2.5 |
| Spring Cloud | 2023.0.1 |
| Netflix Eureka Server | — |
| Spring Boot Actuator | — |

---

## Puerto

| Servicio | Puerto |
|---|---|
| discovery-service | **8761** |

---

## Orden de arranque

```
discovery-service (8761) → config-server (8888) → api-gateway (8080) → business services
```

> Debe ser el **primer servicio** en inicializarse. No tiene dependencias externas.

---

## Servicios registrados

Una vez que el ecosistema está corriendo, estos servicios aparecen en el dashboard de Eureka:

| Service ID | Puerto |
|---|---|
| `config-server` | 8888 |
| `api-gateway` | 8080 |
| `auth-service` | 8081 |
| `catalog-service` | 8082 |
| `interaction-service` | 8083 |
| `playlist-service` | 8084 |
| `social-service` | 8085 |

---

## Configuración relevante

```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false   # No se registra a sí mismo
    fetch-registry: false          # No necesita fetchear el registro
  server:
    enable-self-preservation: false   # Desactivado en dev — activar en producción
    eviction-interval-timer-in-ms: 5000
```

> **Nota de producción:** `enable-self-preservation: false` es apropiado en desarrollo para que los servicios caídos se evicten rápidamente. En producción debe activarse (`true`) para evitar la eliminación masiva de instancias ante pérdidas de red temporales.

---

## Estructura del proyecto

```
discovery-service/
├── src/
│   ├── main/
│   │   ├── java/com/rubymusic/discovery/
│   │   │   └── DiscoveryServiceApplication.java   ← @EnableEurekaServer
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       └── java/com/rubymusic/discovery/
│           └── DiscoveryServiceApplicationTests.java
└── pom.xml
```

---

## Build & Run

```bash
# Build
mvn clean package -DskipTests

# Run
mvn spring-boot:run

# Test
mvn test -Dtest=DiscoveryServiceApplicationTests
```

---

## Dashboard y endpoints

```
# Dashboard web de Eureka (lista de servicios registrados)
GET http://localhost:8761

# Actuator
GET http://localhost:8761/actuator/health
GET http://localhost:8761/actuator/info

# API REST del registro (JSON)
GET http://localhost:8761/eureka/apps
GET http://localhost:8761/eureka/apps/{SERVICE-ID}
```

---

## Variables de entorno

Este servicio no requiere variables de entorno externas. Toda su configuración está en `application.yml` y no maneja credenciales ni secretos.
