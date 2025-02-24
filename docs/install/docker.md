# Docker


### Prerequisites

- Docker


```yaml
services:
  webhookx-migration:
    image: "webhookx/webhookx:latest"
    container_name: webhookx-migration
    environment:
      WEBHOOKX_DATABASE_HOST: webhookx-database
    command: webhookx migrations up
    depends_on:
      webhookx-database:
        condition: service_healthy

  webhookx:
    image: "webhookx/webhookx:latest"
    container_name: webhookx
    environment:
      WEBHOOKX_DATABASE_HOST: webhookx-database
      WEBHOOKX_REDIS_HOST: redis
      WEBHOOKX_ADMIN_LISTEN: 0.0.0.0:8080
      WEBHOOKX_WORKER_ENABLED: true
      WEBHOOKX_PROXY_LISTEN: 0.0.0.0:8081
    ports:
      - "8080:8080"
      - "8081:8081"
    depends_on:
      webhookx-database:
        condition: service_healthy
      redis:
        condition: service_started
      webhookx-migration:
        condition: service_started

  webhookx-database:
    image: postgres:13
    environment:
      POSTGRES_DB: webhookx
      POSTGRES_USER: webhookx
      POSTGRES_HOST_AUTH_METHOD: trust
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}"]
      interval: 3s
      timeout: 5s
      retries: 3
    volumes:
      - "postgres_data:/var/lib/postgresql/data/"

  redis:
    image: redis:6
    command: "--appendonly yes --appendfsync everysec"
    volumes:
      - "redis_data:/data"

volumes:
  postgres_data:
  redis_data:
```


```
$ docker compose up
```

```
$ curl http://localhost:8080
```
