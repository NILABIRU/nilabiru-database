# Nilabiru Database

A self-hosted database stack for the Nilabiru ecosystem, bundling MariaDB, PostgreSQL, and MongoDB into a single Docker Compose setup with a simple deploy script.

---

## Overview

**Nilabiru Database** provisions and manages a compact set of database services — a MySQL-compatible relational database, a PostgreSQL relational database, and a document-oriented NoSQL database — all running in isolated Docker containers. Every published port is bound to the server's Tailscale IP, so the databases are reachable only over your private Tailscale network. Deployment is handled by a single `deploy.sh` script.

---

## Services

| Service               | Image                | Port(s) | Description                                                                        |
| --------------------- | -------------------- | ------- | ---------------------------------------------------------------------------------- |
| **nilabiru-mariadb**  | `mariadb:11.4`       | `3306`  | MySQL-compatible relational database with root password and an additional app user |
| **nilabiru-postgres** | `postgres:17-alpine` | `5432`  | Relational database with configurable user, password, and database name            |
| **nilabiru-mongodb**  | `mongo:8.0.11`       | `27017` | Document-oriented NoSQL database with root authentication                          |

All services run on the default Docker Compose network and use `restart: unless-stopped`. Every published port is bound to the Tailscale IP (`TAILSCALE_IP`) — none are exposed on public network interfaces.

---

## Requirements

- Docker Engine `20.10+`
- Docker Compose `v2+`
- Tailscale installed and connected on the server and all client machines

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/NILABIRU/nilabiru-database.git
cd nilabiru-database
```

### 2. Configure environment variables

Copy the provided `env` file and fill in all values:

```bash
cp env .env
```

Then edit `.env`:

```env
# Tailscale
TAILSCALE_IP=

# MariaDB
MARIADB_ROOT_PASSWORD=
MARIADB_USER=
MARIADB_PASSWORD=

# PostgreSQL
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=

# MongoDB
MONGO_ROOT_USERNAME=
MONGO_ROOT_PASSWORD=
```

### 3. Start the stack

The recommended way is the provided deploy script:

```bash
chmod +x deploy.sh
./deploy.sh
```

`deploy.sh` stops on the first error (`set -e`) and does the following:

1. Validates the Compose configuration with `docker compose config --quiet`.
2. Deploys/redeploys all services with `docker compose up -d --remove-orphans --build`.
3. Always runs a cleanup on exit (even if a step fails) that removes dangling images with `docker image prune -f`.

Alternatively, you can start the stack directly:

```bash
docker compose up -d
```

To verify all services are running:

```bash
docker compose ps
```

---

## Service Access

All services are accessible only via the Tailscale IP of the server.

| Service    | Address                |
| ---------- | ---------------------- |
| MariaDB    | `<TAILSCALE_IP>:3306`  |
| PostgreSQL | `<TAILSCALE_IP>:5432`  |
| MongoDB    | `<TAILSCALE_IP>:27017` |

Example connections:

```bash
# MariaDB
mysql -h <TAILSCALE_IP> -P 3306 -u <MARIADB_USER> -p

# PostgreSQL
psql -h <TAILSCALE_IP> -p 5432 -U <POSTGRES_USER> -d <POSTGRES_DB>

# MongoDB
mongosh "mongodb://<MONGO_ROOT_USERNAME>:<MONGO_ROOT_PASSWORD>@<TAILSCALE_IP>:27017"
```

---

## Data Persistence

All stateful services use Docker named volumes for reliable persistence across restarts and redeployments.

| Volume          | Type         | Service    | Container path             |
| --------------- | ------------ | ---------- | -------------------------- |
| `mariadb-data`  | Named volume | MariaDB    | `/var/lib/mysql`           |
| `postgres-data` | Named volume | PostgreSQL | `/var/lib/postgresql/data` |
| `mongodb-data`  | Named volume | MongoDB    | `/data/db`                 |

> **Note:** The credentials in `.env` are only applied on the **first** startup, when a volume is still empty. Changing them later will not update existing databases — change them inside the database itself, or remove the volume (this deletes all data).

---

## License

This project is licensed under the [MIT License](LICENSE).
Copyright © 2026 Andry Pebrianto
