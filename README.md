# Configuración de Movie, Booking y Gateway

Config Server sirve `movie-service.yml`, `booking-service.yml` y `gateway-service.yml`. Los archivos `*-docker.yml` se aplican con el perfil `docker` (`SPRING_PROFILES_ACTIVE=docker`). Cada aplicación conserva localmente solo `spring.application.name` y `spring.config.import`.

Arranca Config Server antes que los clientes. En local, los clientes usan `http://localhost:50000` por defecto. En Docker, proporciona `CONFIG_SERVER_URL=http://config-service:50000` a Movie, Booking y Gateway.

Variables necesarias para PostgreSQL:

| Servicio | Variables |
| --- | --- |
| Movie | `MOVIE_DB_PASSWORD`; en Docker también `MOVIE_DB_URL` y `MOVIE_DB_USERNAME` |
| Booking | `BOOKING_DB_PASSWORD`; en Docker también `BOOKING_DB_URL` y `BOOKING_DB_USERNAME` |

Variables opcionales: `MOVIE_PORT` (51100), `BOOKING_PORT` (51200), `GATEWAY_PORT` (8080), `MOVIE_SERVICE_INSTANCE_URL` (instancia que Booking usa mediante LoadBalancer), `GATEWAY_MOVIE_URI` y `GATEWAY_BOOKING_URI` (destinos del Gateway). Los perfiles Docker usan los nombres de servicio `movie-service` y `booking-service` como valores predeterminados.

El Config Server del proyecto apunta normalmente al repositorio GitHub. Los cambios locales de este repositorio deben publicarse allí para que ese servidor los sirva. Para probar el repositorio local sin publicarlo, arranca Config Server con el perfil `native` y `spring.cloud.config.server.native.search-locations=file:/home/rob/projects/exam-mitocode-config-repo`.
