# Northwind database for Postgres

A simple sql script that will populate a database with the famous northwind example, adapted for postgres.

<img src=ER.png />

## Getting started:

### Manually

Use the provided sql file `northwind.sql` in order to populate your database.

### With Docker and Docker Compose

#### Prerequisites: install Docker and Docker Compose (v2)

 https://www.docker.com/get-started

 https://docs.docker.com/compose/install/

#### 1. Start the database

```bash
docker compose up -d
```

The first start pulls the `postgres:18` image and loads `northwind.sql`, which takes a few seconds. Check that the container is ready:

```bash
docker compose ps
```

Wait until the status is `Up (healthy)`.

PostgreSQL is exposed on **port 5434** of your machine, bound to `127.0.0.1` only (not reachable from other hosts). Inside the container it still listens on the standard port 5432.

| Setting  | Value       |
|----------|-------------|
| Host     | `127.0.0.1` |
| Port     | `5434`      |
| Database | `northwind` |
| User     | `postgres`  |
| Password | `north`     |

#### 2. Run a psql client

##### Method 1: Via the docker compose container

```bash
docker compose exec postgres psql -U postgres -d northwind
```

```
northwind=# select * from us_states;
 state_id |      state_name      | state_abbr | state_region
----------+----------------------+------------+--------------
        1 | Alabama              | AL         | south
        2 | Alaska               | AK         | north
        ...
```

Alternatively, you can launch bash, then psql:

```bash
docker compose exec postgres /bin/bash

# You are now inside the "northwind" docker container
psql -U postgres northwind
```

##### Method 2: Direct access via port 5434

If you have psql on your path, run this directly from your computer (it connects to the docker database):

```bash
psql -h 127.0.0.1 -p 5434 -U postgres -d northwind
# password: north
```

The same host, port and credentials work for GUI clients such as DataGrip, DBeaver or pgAdmin installed on your machine.

#### 3. Stop the database

```bash
docker compose down
```

This removes the container but keeps the data (see below).

#### 4. Files & persistence

Your modifications to the postgres database(s) are persisted in the `postgresql_data` docker volume, and are available again after the next `docker compose up -d`.

`northwind.sql` is loaded only when the volume is empty (first start). If you need to delete the database data, run `docker compose down -v`; the database will then be repopulated from scratch on the next `docker compose up -d`.

If you need to upload any files into your db container, just copy them to the `files` local folder. They will be available at the `/files` path inside the container.

#### Troubleshooting

- **`port is already allocated`**: another process uses port 5434. Change the left side of `127.0.0.1:5434:5432` in `docker-compose.yml`.
- **Password change has no effect**: `POSTGRES_PASSWORD` is applied only on the first initialization. Run `docker compose down -v` to recreate the database.
