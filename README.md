# Dockerfile for pg_ivm

Dockerfile for extension pg_ivm.

This image based on [postgres](https://hub.docker.com/_/postgres/) and could use as same as postgres images.

[日本語版 README](./README.ja.md)

## Quick start

```
docker run --name postgres -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 aeffix/pg_ivm:postgres18.4-pg_ivm1.15
```

## Available image tags

- `aeffix/pg_ivm:postgres16.14-pg_ivm1.15`
- `aeffix/pg_ivm:postgres17.10-pg_ivm1.15`
- `aeffix/pg_ivm:postgres18.4-pg_ivm1.15`

Major-version tags can also be published for convenience:

- `aeffix/pg_ivm:postgres16`
- `aeffix/pg_ivm:postgres17`
- `aeffix/pg_ivm:postgres18`

## Try pg_ivm quickly

You can test `pg_ivm` with a simple JSON-based example after the container starts.

```bash
docker exec -it postgres psql -U postgres -d postgres
```

```sql
CREATE EXTENSION IF NOT EXISTS pg_ivm;
DROP TABLE IF EXISTS ivm_orders;
DROP TABLE IF EXISTS raw_orders;

CREATE TABLE raw_orders (payload jsonb NOT NULL);

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

- `TABLE ivm_orders;` returns 2 rows extracted from JSON
- after the last `INSERT`, `row_count` becomes `3`
- `total_amount` becomes `2650`

## How to use

----
Create docker-compose.yml with lines below.
```
version: "3.8"

services:
  db:
    container_name: pg_ivm
    image: aeffix/pg_ivm:postgres18.4-pg_ivm1.15
    ports:
      -   5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    restart: always
```

now You can run `docker-compose up -d` to start service.
by use any data manage tool to connect to database `postgres`.
You can use pg_ivm extension.

please take a look at [pg_ivm website](https://github.com/sraoss/pg_ivm) for details.
