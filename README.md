# Configuración de Auth, Movie, Booking y Gateway

Config Server sirve `auth-service.yml`, `movie-service.yml`, `booking-service.yml` y `gateway-service.yml`. Los archivos `*-docker.yml` se aplican con el perfil `docker` (`SPRING_PROFILES_ACTIVE=docker`). Cada aplicación conserva localmente solo `spring.application.name` y `spring.config.import`.

Arranca Config Server antes que los clientes. En local, los clientes usan `http://localhost:50000` por defecto. En Docker, proporciona `CONFIG_SERVER_URL=http://config-service:50000` a Auth, Movie, Booking y Gateway.

Variables necesarias para PostgreSQL:

| Servicio | Variables |
| --- | --- |
| Movie | `MOVIE_DB_PASSWORD`; en Docker también `MOVIE_DB_URL` y `MOVIE_DB_USERNAME` |
| Booking | `BOOKING_DB_PASSWORD`; en Docker también `BOOKING_DB_URL` y `BOOKING_DB_USERNAME` |

Variables opcionales: `AUTH_PORT` (50003), `MOVIE_PORT` (51100), `BOOKING_PORT` (51200), `GATEWAY_PORT` (8080), `MOVIE_SERVICE_INSTANCE_URL` (instancia que Booking usa mediante LoadBalancer), `GATEWAY_AUTH_URI`, `GATEWAY_MOVIE_URI` y `GATEWAY_BOOKING_URI` (destinos del Gateway). Los perfiles Docker usan los nombres de servicio correspondientes como valores predeterminados.

Auth emite JWT RS256. `AUTH_JWT_ISSUER` (predeterminado `http://localhost:50003`) debe coincidir en Auth y Gateway; `AUTH_JWT_ACCESS_TTL_SECONDS` vale 3600 por defecto. Gateway valida la firma con `AUTH_JWKS_URI` (predeterminado `http://localhost:50003/oauth2/jwks`, o `http://auth-service:50003/oauth2/jwks` en Docker). Registro, login, JWKS y Swagger son públicos en Gateway; las demás rutas requieren Bearer JWT. Movie y Booking todavía deben validar JWT en sus propios puertos para impedir el acceso directo sin token.

Booking revisa cada minuto las reservas `PENDING` con más de 10 minutos, las cancela y solicita a Movie liberar el asiento por `bookingId`. Se puede ajustar con `BOOKING_RECOVERY_INITIAL_DELAY_MS`, `BOOKING_RECOVERY_INTERVAL_MS` y `BOOKING_RECOVERY_STALE_AFTER_MS`. Si PostgreSQL sigue caído, la revisión se reintenta en el siguiente ciclo; no se puede garantizar la cancelación mientras la base esté indisponible. Una liberación remota fallida queda registrada para seguimiento operativo.

El Config Server del proyecto apunta normalmente al repositorio GitHub. Los cambios locales de este repositorio deben publicarse allí para que ese servidor los sirva. Para probar el repositorio local sin publicarlo, arranca Config Server con el perfil `native` y `spring.cloud.config.server.native.search-locations=file:/home/rob/projects/exam-mitocode-config-repo`.
