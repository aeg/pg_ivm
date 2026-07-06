# Dockerfile for pg_ivm

Docker image for PostgreSQL with the `pg_ivm` extension.

This image is based on the official [postgres](https://hub.docker.com/_/postgres/) image and can be used in the same way as standard PostgreSQL images.

## Supported tags

Fixed tags:

- `aeffix/pg_ivm:postgres16.14-pg_ivm1.15`
- `aeffix/pg_ivm:postgres17.10-pg_ivm1.15`
- `aeffix/pg_ivm:postgres18.4-pg_ivm1.15`

Major-version tags:

- `aeffix/pg_ivm:postgres16`
- `aeffix/pg_ivm:postgres17`
- `aeffix/pg_ivm:postgres18`

`latest`:

- `aeffix/pg_ivm:latest` = `aeffix/pg_ivm:postgres18.4-pg_ivm1.15`

## Quick start

```bash
docker run --name postgres -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 aeffix/pg_ivm:postgres18.4-pg_ivm1.15
```

## Try pg_ivm quickly

```bash
docker exec -it postgres psql -U postgres -d postgres
```

```sql
CREATE EXTENSION IF NOT EXISTS pg_ivm;
DROP TABLE IF EXISTS ivm_orders;
DROP TABLE IF EXISTS raw_orders;

CREATE TABLE raw_orders (
    payload jsonb NOT NULL
);

INSERT INTO raw_orders (payload) VALUES
    ('{"id": 1001, "customer": "alice", "amount": 1200, "status": "new"}'),
    ('{"id": 1002, "customer": "bob", "amount": 800, "status": "paid"}');

SELECT pgivm.create_immv(
    'ivm_orders',
    $$SELECT
        (payload->>'id')::int AS order_id,
        payload->>'customer' AS customer,
        (payload->>'amount')::int AS amount,
        payload->>'status' AS status
    FROM raw_orders$$
);

TABLE ivm_orders;

INSERT INTO raw_orders (payload)
VALUES ('{"id": 1003, "customer": "carol", "amount": 650, "status": "shipped"}');

SELECT count(*) AS row_count, sum(amount) AS total_amount FROM ivm_orders;
```

Expected result:

- `TABLE ivm_orders;` returns 2 rows
- after the last `INSERT`, `row_count` becomes `3`
- `total_amount` becomes `2650`

## How to use

Create `docker-compose.yml` with the following content.

```yaml
version: "3.8"

services:
  db:
    container_name: pg_ivm
    image: aeffix/pg_ivm:postgres18.4-pg_ivm1.15
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    restart: always
```

Then run:

```bash
docker-compose up -d
```

You can connect to the `postgres` database with your preferred database tool and use the `pg_ivm` extension.

For details, see the official `pg_ivm` repository: https://github.com/sraoss/pg_ivm
