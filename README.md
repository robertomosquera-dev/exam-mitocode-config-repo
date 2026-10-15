# Configuración de Auth, Movie, Booking y Gateway

Config Server sirve `auth-service.yml`, `movie-service.yml`, `booking-service.yml` y `gateway-service.yml`. Los archivos `*-docker.yml` se aplican con el perfil `docker` (`SPRING_PROFILES_ACTIVE=docker`). Cada aplicación conserva localmente solo `spring.application.name` y `spring.config.import`.

Arranca Config Server antes que los clientes. En local, los clientes usan `http://localhost:50000` por defecto. En Docker, proporciona `CONFIG_SERVER_URL=http://config-service:50000` a Auth, Movie, Booking y Gateway.

Variables necesarias para PostgreSQL:

| Servicio | Variables |
| --- | --- |
| Movie | `MOVIE_DB_PASSWORD`; en Docker también `MOVIE_DB_URL` y `MOVIE_DB_USERNAME` |
| Booking | `BOOKING_DB_PASSWORD`; en Docker también `BOOKING_DB_URL` y `BOOKING_DB_USERNAME` |

Variables opcionales: `AUTH_PORT` (50003), `MOVIE_PORT` (51100), `BOOKING_PORT` (51200), `GATEWAY_PORT` (8080), `MOVIE_SERVICE_INSTANCE_URL` (instancia que Booking usa mediante LoadBalancer), `GATEWAY_AUTH_URI`, `GATEWAY_MOVIE_URI` y `GATEWAY_BOOKING_URI` (destinos del Gateway). Los perfiles Docker usan los nombres de servicio correspondientes como valores predeterminados.

Auth emite JWT RS256. `AUTH_JWT_ISSUER` (predeterminado `http://localhost:50003`) debe coincidir en Auth, Gateway, Movie y Booking; `AUTH_JWT_ACCESS_TTL_SECONDS` vale 3600 por defecto. Los tres validadores obtienen la llave pública mediante `AUTH_JWKS_URI` (predeterminado `http://localhost:50003/oauth2/jwks`, o `http://auth-service:50003/oauth2/jwks` en los perfiles Docker de Movie y Booking). Registro, login, JWKS y Swagger son públicos en Gateway. Movie y Booking permiten Swagger, OpenAPI y Actuator sin token; todas sus rutas de negocio, incluidas consultas, requieren Bearer JWT incluso por acceso directo. Booking reenvía el Bearer de la solicitud actual a Movie durante la Saga.

Auth genera su par RSA en memoria al iniciar. Reiniciarlo invalida los tokens anteriores; vuelve a iniciar sesión para obtener uno nuevo. Auth no emite `aud`, por lo que los servicios validan firma RS256, `iss` y vigencia, sin exigir una audiencia.

Booking revisa cada minuto las reservas `PENDING` con más de 10 minutos y las cancela. Se puede ajustar con `BOOKING_RECOVERY_INITIAL_DELAY_MS`, `BOOKING_RECOVERY_INTERVAL_MS` y `BOOKING_RECOVERY_STALE_AFTER_MS`. Si PostgreSQL sigue caído, la revisión se reintenta en el siguiente ciclo; no se puede garantizar la cancelación mientras la base esté indisponible. La tarea programada no tiene un Bearer de usuario: su intento de liberar un asiento en Movie recibirá 401 hasta que se defina una credencial de servicio. El error queda registrado para seguimiento operativo; la cancelación local persiste.

El Config Server del proyecto apunta normalmente al repositorio GitHub. Los cambios locales de este repositorio deben publicarse allí para que ese servidor los sirva. Para probar el repositorio local sin publicarlo, arranca Config Server con el perfil `native` y `spring.cloud.config.server.native.search-locations=file:/home/rob/projects/exam-mitocode-config-repo`.
