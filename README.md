# Post-contenido — Unidad 5: Integración en Aplicaciones Web

## Descripción

Repositorio del post-contenido de la Unidad 5 de Patrones de Diseño de Software. Contiene un proyecto Spring Boot para la gestión de reservas de laboratorios de cómputo, desarrollado en dos partes: una API REST organizada por capas y una interfaz web MVC con Thymeleaf que reutiliza la misma lógica de negocio.

## Parte 1 — Repository, Service y Controller REST

La capa de persistencia utiliza `LaboratorioRepository` y `ReservaRepository`, interfaces que extienden `JpaRepository`. `ReservaRepository` incorpora una consulta JPQL para detectar reservas que se solapan en un mismo laboratorio.

`ReservaService` concentra las reglas de negocio relacionadas con la disponibilidad, el horario de atención, la duración de las reservas y su cancelación. Los controladores `LaboratorioController` y `ReservaController` exponen los endpoints REST bajo `/api/laboratorios` y `/api/reservas`.

La aplicación utiliza H2 como base de datos en memoria. Las entidades y los repositorios se encuentran en `model/` y `repository/`; la lógica de negocio, en `service/`; el manejo de errores, en `exception/`; y los endpoints REST, en `controller/`.

## Parte 2 — Vista MVC con Thymeleaf

`ReservaWebController` expone la interfaz web bajo `/reservas`, con una vista para listar las reservas y otra para registrar nuevas. El controlador inyecta la misma clase `ReservaService` utilizada por la API REST, evitando duplicar las reglas de negocio.

Las plantillas `templates/reservas/lista.html` y `templates/reservas/nueva.html` permiten consultar, crear y cancelar reservas desde el navegador.

`ReservaWebExceptionHandler` maneja las excepciones de dominio utilizadas por el servicio y presenta sus mensajes mediante redirecciones y atributos flash. De esta manera, la API REST conserva sus respuestas JSON y la interfaz MVC presenta mensajes comprensibles para el usuario.

## Cómo ejecutar

Desde la raíz del proyecto:

```bash
mvn clean package
mvn spring-boot:run
```

También se puede utilizar el Maven Wrapper incluido en el repositorio:

```bash
./mvnw clean package
./mvnw spring-boot:run
```

En Windows PowerShell:

```powershell
.\mvnw.cmd clean package
.\mvnw.cmd spring-boot:run
```

* API REST de reservas: `http://localhost:8080/api/reservas`
* API REST de laboratorios: `http://localhost:8080/api/laboratorios`
* Interfaz web: `http://localhost:8080/reservas`
* Formulario de reserva: `http://localhost:8080/reservas/nueva`

## Decisiones de diseño

### Punto de decisión 1 — Ubicación de la validación de solapamiento

La consulta que identifica las reservas que se cruzan en un mismo laboratorio se encuentra en `ReservaRepository`, porque requiere consultar los datos persistidos. La decisión de aceptar o rechazar una reserva se encuentra en `ReservaService`, que utiliza el resultado de la consulta para aplicar la regla de negocio.

Si el controlador llamara directamente a `buscarSolapamientos()`, asumiría responsabilidades propias de la lógica de negocio y acoplaría la presentación con la persistencia. Mantener la decisión en el servicio permite que tanto REST como MVC apliquen la misma regla.

### Punto de decisión 2 — Reglas con y sin apoyo del Repository

La detección de solapamientos necesita consultar las reservas existentes, por lo que requiere el apoyo del repositorio. En cambio, las validaciones del horario de atención y de la duración se resuelven con los valores de inicio y fin recibidos, sin consultar la base de datos.

Por esta razón, estas reglas se implementan en `ReservaService`: centraliza las validaciones y evita que los controladores tengan que conocer o duplicar su funcionamiento.

### Punto de decisión 3 — Cómo comparten Service el Controller MVC y el REST

`ReservaController` y `ReservaWebController` reciben `ReservaService` mediante inyección de dependencias. Spring administra el servicio como un bean compartido, por lo que ambas superficies utilizan la misma implementación de las reglas de negocio.

Crear un segundo servicio para MVC duplicaría la lógica de validación y podría producir comportamientos diferentes si una regla se modifica en una sola de las implementaciones. La reutilización mantiene la consistencia y facilita el mantenimiento.

### Punto de decisión 4 — Manejo de errores consistente entre MVC y REST

Ambas superficies utilizan las excepciones de dominio `ReservaConflictException` y `RecursoNoEncontradoException`, pero presentan los errores de manera diferente. `GlobalRestExceptionHandler` gestiona las respuestas de la API REST, mientras que `ReservaWebExceptionHandler`, restringido a `ReservaWebController` mediante `assignableTypes`, utiliza redirecciones y atributos flash para mostrar los mensajes en las vistas Thymeleaf.

No se utiliza un único `@RestControllerAdvice` para ambas superficies porque su comportamiento está orientado a respuestas REST, mientras que MVC necesita resolver una vista o redirigir al navegador. Separar los manejadores por superficie evita mezclar responsabilidades y conserva el mismo vocabulario de errores del dominio.

## Herramientas utilizadas

* Java 17
* Spring Boot 3.2
* Spring Data JPA
* H2 Database
* Thymeleaf
* Apache Maven
* Postman y PowerShell para pruebas de endpoints
* Git y GitHub

## Conclusiones

El desarrollo permitió aplicar una arquitectura por capas, separando la persistencia, las reglas de negocio y la presentación. La principal decisión consistió en ubicar cada regla en la capa adecuada, diferenciando las consultas que requieren acceso a los datos de las validaciones que pueden resolverse en el servicio. La incorporación de Thymeleaf permitió reutilizar la lógica existente sin duplicar el servicio. Finalmente, se mantuvo un manejo de errores coherente en REST y MVC, adaptando la respuesta a las necesidades de cada interfaz.


## Autor

Eileen Balaguera Rodriguez

Ingeniería de Sistemas — Universidad de Santander (UDES)
