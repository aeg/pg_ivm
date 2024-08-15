# Dockerfile for pg_ivm

Dockerfile for extension pg_ivm.

This image based on [postgres](https://hub.docker.com/_/postgres/) and could use as same as postgres images.

## Quick start

```
docker run --name postgres -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 aeffix/pg_ivm:postgres17beta3-pg_ivm1.9
```

## How to use

----
Create docker-compose.yml with lines below.
```
version: "3.8"

services:
  db:
    container_name: pg_ivm
    image: aeffix/pg_ivm:postgres17beta3-pg_ivm1.9
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