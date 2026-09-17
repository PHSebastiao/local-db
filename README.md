# Local Dev Databases

Persistent local databases in Docker. Data lives in named volumes, so it survives
restarts and reboots.

## Services

**Enabled by default**

| Database | Version | Port | User | Password |
|---|---|---|---|---|
| MongoDB | 8.0 | `27017` | `root` | `root` |
| MySQL | 8.4 | `3306` | `root` | `root` |
| PostgreSQL | 16 | `5432` | `root` | `root` |
| Redis | 8 | `6379` | — | — |

**Opt-in** — add the profile name to `COMPOSE_PROFILES` in `.env`

| Database | Version | Port | User | Password | Profile |
|---|---|---|---|---|---|
| MariaDB | 11.8 | `3307` | `root` | `root` | `mariadb` |
| Elasticsearch | 8.19 | `9200` | — | — | `elasticsearch` |
| VersityGW (S3) | 1.8 | `7070`, `7071` | `root` | `root` | `versitygw` |
| ClickHouse | 26.8 | `8123`, `9005` | `default` | `root` | `clickhouse` |

## Setup

Needs Docker Desktop, or Docker Engine with the Compose v2 plugin.

```bash
git clone https://github.com/PHSebastiao/local-db.git
cd local-db
docker compose up -d
```

## Toggling services

`.env` decides which profiles start. Dropping a service leaves its data volume intact.

```bash
COMPOSE_PROFILES=mongo,postgres                     # just two
COMPOSE_PROFILES=mongo,mysql,postgres,redis,clickhouse
```

Re-run `docker compose up -d` after editing.

## Connecting

**MongoDB**

```bash
mongosh "mongodb://root:root@localhost:27017"
```

**MySQL**

```bash
mysql -h 127.0.0.1 -P 3306 -u root -proot
```

**MariaDB** — port `3307`, so it can run alongside MySQL

```bash
mysql -h 127.0.0.1 -P 3307 -u root -proot
```

**PostgreSQL**

```bash
PGPASSWORD=root psql -h localhost -U root
```

**Redis**

```bash
redis-cli
```

**Elasticsearch** — no auth, single-node, localhost only

```bash
curl localhost:9200
```

**VersityGW** — S3 API on `7070`, web console at <http://localhost:7071>

```bash
export AWS_ACCESS_KEY_ID=root AWS_SECRET_ACCESS_KEY=root
aws --endpoint-url http://localhost:7070 s3 mb s3://mybucket
aws --endpoint-url http://localhost:7070 s3 cp file.txt s3://mybucket/
```

Point any AWS SDK at `http://localhost:7070` with path-style addressing.

**ClickHouse** — HTTP on `8123`, native protocol on `9005`

```bash
curl 'http://localhost:8123/' --data 'SELECT version()' --user default:root
```

## Managing

```bash
docker compose ps -a          # status, including disabled services
docker compose logs -f postgres
docker compose down           # stop, keep all data
docker compose down -v        # stop and delete all data — destructive
```

Reset a single database:

```bash
docker compose rm -sf postgres
docker volume rm db_postgres-data
docker compose up -d
```