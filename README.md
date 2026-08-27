# docker-config

A collection of Docker Compose configurations for local supporting services: MySQL, PostgreSQL, Redis, MinIO, and n8n. Each service has its own Compose file, so it can be started independently as needed.

> These configurations are intended for development or internal use. Do not use the included credentials or default exposed ports in a production environment that is reachable from the internet.

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or Docker Engine with Docker Compose, running on your machine.
- Docker Compose v2.
- The following host ports must be available: `3306`, `5432`, `6379`, `9000`, `9001`, and `5678`.

Check your Docker Compose installation:

```bash
docker compose version
```

## Repository contents

| Service | Compose file | Host ports | Description |
| --- | --- | --- | --- |
| MySQL | `docker-compose.mysql.yml` | `3306` | MySQL database with a `testing` database initializer. |
| PostgreSQL | `docker-compose.pgsql.yml` | `5432` | PostgreSQL database with a `testing` database initializer. |
| Redis | `docker-compose.redis.yml` | `6379` | Redis key-value store/cache. |
| MinIO | `docker-compose.minio.yml` | `9000`, `9001` | S3-compatible object storage and a `silog` bucket initializer. |
| n8n | `docker-compose.n8n.yml` | `5678` | n8n workflow automation platform. |

Supporting directories:

- `mysql/create-testing-database.sh` - creates the `testing` database and grants access to the MySQL application user on first initialization.
- `pgsql/create-testing-database.sql` - creates the `testing` database if it does not already exist.
- `pgsql/postgresql.conf` - additional PostgreSQL settings (`max_connections` and `max_locks_per_transaction`).
- `redis/redis.conf` - Redis configuration template.

## Starting services

Run these commands from the repository root. Start one or more services as needed:

```bash
docker compose -f docker-compose.mysql.yml up -d
docker compose -f docker-compose.pgsql.yml up -d
docker compose -f docker-compose.redis.yml up -d
docker compose -f docker-compose.minio.yml up -d
docker compose -f docker-compose.n8n.yml up -d
```

Check status and follow logs, for example for PostgreSQL:

```bash
docker compose -f docker-compose.pgsql.yml ps
docker compose -f docker-compose.pgsql.yml logs -f
```

Stop a service without removing its data:

```bash
docker compose -f docker-compose.pgsql.yml down
```

Replace the Compose filename with the service you want to manage.

## Service access

### MySQL

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `3306` |
| Root password | `rootPassword*01` |
| Application user | `mysqluser` |
| Application password | `secret*098` |
| Initial database | `testing_db` |
| Additional database | `testing` |

Connect from the container:

```bash
docker exec -it mysql-local mysql -u mysqluser -p testing
```

### PostgreSQL

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| User | `root` |
| Password | `secret*098` |
| Initial database | `test_db` |
| Additional database | `testing` |

Connect from the container:

```bash
docker exec -it pgsql-local psql -U root -d testing
```

### Redis

Redis is exposed at `localhost:6379` and no password is defined by the current Compose configuration.

```bash
docker exec -it redis-db redis-cli ping
```

### MinIO

- S3 API: `http://localhost:9000`
- Console: `http://localhost:9001`
- User/access key: `root`
- Password/secret key: `secret*098`
- Bucket initialized by the configuration: `silog` (made public by the `createbuckets` container)

The bucket initializer waits 10 seconds, runs once, and then exits. If MinIO was not ready at that point, rerun the initializer:

```bash
docker compose -f docker-compose.minio.yml up createbuckets
```

### n8n

Open `http://localhost:5678`. The current configuration enables basic authentication with the `root` user; its password is defined directly in `docker-compose.n8n.yml`. The `n8nvol` volume is declared as external, so it must exist before starting the service:

```bash
docker volume create n8nvol
docker compose -f docker-compose.n8n.yml up -d
```

Before using this configuration on another machine or server, change `N8N_HOST` from the IP address in the Compose file to the target hostname or domain.

## Persistent data and initialization

MySQL, PostgreSQL, Redis, MinIO, and n8n store data in Docker volumes. The SQL and shell initialization scripts run only when the respective database volume is new or empty. Editing an initializer after the database has been created does not apply the changes automatically.

To reset a service's data, stop the service and remove its volume. The following command permanently deletes data:

```bash
docker compose -f docker-compose.mysql.yml down -v
```

Use this carefully and only for services whose data may be removed.

## Configuration notes

- Images use the `latest` tag, so service versions may change when images are pulled again. Pin images to specific versions for stable environments.
- `postgresql.conf` and `redis.conf` are mounted into their containers, but they are not used at startup because the activating `command`/`entrypoint` lines are currently commented out in the Compose files. Uncomment those lines to enable the custom configuration.
- Service ports are published to the host. Restrict access with a firewall, or avoid publishing ports that are unnecessary in production.
- Change all passwords, the n8n host setting, and authentication settings before using these files outside a local or internal environment.

## Validate the configuration

Docker Compose can validate syntax and resolve configuration without starting containers:

```bash
docker compose -f docker-compose.mysql.yml config --quiet
docker compose -f docker-compose.pgsql.yml config --quiet
docker compose -f docker-compose.redis.yml config --quiet
docker compose -f docker-compose.minio.yml config --quiet
docker compose -f docker-compose.n8n.yml config --quiet
```
