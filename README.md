# Reservas Labs API

API REST desarrollada con Spring Boot para la gestión de reservas de laboratorios universitarios.

El proyecto implementa una arquitectura en capas, utilizando Spring Data JPA para el acceso a datos y H2 como base de datos en memoria.

## Tecnologías utilizadas

* Java 17
* Spring Boot 3.2.x
* Maven
* Spring Web
* Spring Data JPA
* H2 Database
* Lombok
* Jakarta Validation

## Estructura del proyecto

```text
reservas-labs-api/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/universidad/reservaslabs/
│   │   │       ├── model/
│   │   │       ├── repository/
│   │   │       ├── service/
│   │   │       ├── exception/
│   │   │       ├── controller/
│   │   │       └── ReservasLabsApiApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
├── pom.xml
├── mvnw
├── mvnw.cmd
└── .mvn/
```

## Requisitos

Para ejecutar el proyecto se requiere:

* Java JDK 17
* Maven o Maven Wrapper
* Git

Verificar la versión de Java:

```powershell
java -version
```

## Ejecución del proyecto

Desde la carpeta raíz del proyecto se puede utilizar el Maven Wrapper incluido:

```powershell
.\mvnw.cmd spring-boot:run
```

También es posible compilar y ejecutar las pruebas mediante:

```powershell
.\mvnw.cmd clean test
```

Cuando la aplicación se inicia correctamente, queda disponible en:

```text
http://localhost:8080
```

## Base de datos H2

El proyecto utiliza H2 como base de datos en memoria.

La consola H2 está disponible en:

```text
http://localhost:8080/h2-console
```

Configuración:

```text
JDBC URL: jdbc:h2:mem:reservas_labs_db
Usuario: sa
Contraseña:
```

La configuración utiliza `create-drop`, por lo que los datos se generan durante la ejecución y se eliminan al detener la aplicación.

## Endpoints REST

### Laboratorios

**Listar laboratorios**

```http
GET /api/laboratorios
```

**Obtener laboratorio por ID**

```http
GET /api/laboratorios/{id}
```

**Crear laboratorio**

```http
POST /api/laboratorios
```

Ejemplo:

```json
{
  "nombre": "Laboratorio de Sistemas 1",
  "ubicacion": "Bloque A - 201",
  "capacidad": 30,
  "tipo": "COMPUTO"
}
```

### Reservas

**Listar reservas**

```http
GET /api/reservas
```

**Obtener reserva por ID**

```http
GET /api/reservas/{id}
```

**Consultar reservas por laboratorio**

```http
GET /api/reservas/laboratorio/{laboratorioId}
```

**Crear reserva**

```http
POST /api/reservas
```

Ejemplo:

```json
{
  "laboratorio": {
    "id": 1
  },
  "nombreSolicitante": "Eileen Balaguera",
  "correoSolicitante": "eileen@example.com",
  "inicio": "2026-08-10T09:00:00",
  "fin": "2026-08-10T11:00:00",
  "motivo": "Clase de Ingeniería de Sistemas"
}
```

**Cancelar reserva**

```http
DELETE /api/reservas/{id}
```

## Reglas de negocio

El sistema implementa las siguientes reglas:

1. El laboratorio asociado a una reserva debe existir.
2. La fecha y hora de finalización debe ser posterior a la fecha y hora de inicio.
3. La duración mínima de una reserva es de 30 minutos.
4. La duración máxima es de 3 horas.
5. Las reservas deben realizarse entre las 07:00 y las 21:00.
6. No se permiten reservas solapadas para un mismo laboratorio.
7. Las reservas canceladas no generan conflictos de disponibilidad.
8. Una reserva cuyo horario de inicio ya pasó no puede ser cancelada.

## Decisiones de diseño

### Punto de decisión 1: Arquitectura en capas

Se implementó una arquitectura en capas para separar las responsabilidades de la aplicación.

La capa `controller` recibe las solicitudes HTTP y construye las respuestas. La capa `service` concentra las reglas de negocio relacionadas con las reservas. La capa `repository` gestiona el acceso a la base de datos mediante Spring Data JPA y la capa `model` representa las entidades del dominio.

Esta separación permite mantener una distribución clara de responsabilidades y facilita el mantenimiento y evolución del sistema.

`ReservaController` delega las operaciones que requieren reglas de negocio a `ReservaService`, especialmente la creación y cancelación de reservas.

Por otro lado, `LaboratorioController` utiliza directamente `LaboratorioRepository` para las operaciones CRUD básicas, debido a que estas operaciones no requieren una lógica de negocio adicional.

### Punto de decisión 2: Validación de solapamientos

La disponibilidad de los laboratorios se valida mediante una consulta específica implementada en `ReservaRepository`.

El método:

```java
buscarSolapamientos(
    Long laboratorioId,
    LocalDateTime inicio,
    LocalDateTime fin
)
```

permite consultar directamente las reservas que se intersectan con el intervalo solicitado.

La consulta excluye las reservas cuyo estado sea `CANCELADA`.

La decisión de utilizar esta consulta evita recuperar todas las reservas para realizar la validación en memoria. El repositorio se encarga de consultar los datos y `ReservaService` interpreta el resultado para aplicar la regla de negocio.

Cuando existe una reserva activa que se solapa con el horario solicitado, el servicio genera un `ReservaConflictException`.

## Commits principales

El desarrollo de la Parte 1 se organizó mediante commits descriptivos:

```text
feat: inicializar reservas-labs-api con entidades y repositories

feat(service): implementar ReservaService con reglas de solapamiento y horario

feat(controller): exponer endpoints REST de laboratorios y reservas
```

## Parte 2 — MVC con Thymeleaf

Esta sección será completada al finalizar la Parte 2.

En la Parte 2 se incorporará la interfaz web utilizando Spring MVC y Thymeleaf, manteniendo la separación de responsabilidades definida en la arquitectura del proyecto.

Se documentarán las vistas, controladores MVC, formularios, navegación e integración con la capa de servicio.

## Autor

Eileen Balaguera Rodriguez

Ingeniería de Sistemas — Universidad de Santander (UDES)
