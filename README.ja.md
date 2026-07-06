# pg_ivm 用 Dockerfile

`pg_ivm` 拡張を組み込んだ PostgreSQL 用 Docker image です。

この image は [postgres](https://hub.docker.com/_/postgres/) 公式 image をベースにしており、通常の PostgreSQL image と同様に利用できます。

[English README](./README.md)

## クイックスタート

```bash
docker run --name postgres -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 aeffix/pg_ivm:postgres18.4-pg_ivm1.14
```

## 利用できる image tag

- `aeffix/pg_ivm:postgres16.14-pg_ivm1.15`
- `aeffix/pg_ivm:postgres17.10-pg_ivm1.14`
- `aeffix/pg_ivm:postgres18.4-pg_ivm1.14`

必要であれば、次のようなメジャーバージョン用 tag も公開できます。

- `aeffix/pg_ivm:postgres16`
- `aeffix/pg_ivm:postgres17`
- `aeffix/pg_ivm:postgres18`

## pg_ivm を簡単に試す

コンテナ起動後、次の手順で `pg_ivm` をすぐ試せます。JSON を格納したテーブルを作り、その内容を参照する Incremental Materialized View を作成します。

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

確認できること:

- `TABLE ivm_orders;` で JSON から展開された 2 行を参照できます
- 最後の `INSERT` 実行後、`row_count` は `3` になります
- `total_amount` は `2650` になります

## 使い方

以下の内容で `docker-compose.yml` を作成してください。

```yaml
version: "3.8"

services:
  db:
    container_name: pg_ivm
    image: aeffix/pg_ivm:postgres18.4-pg_ivm1.14
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    restart: always
```

その後、`docker-compose up -d` を実行すると起動できます。任意のデータベース管理ツールで `postgres` データベースへ接続し、`pg_ivm` 拡張を利用できます。

詳しくは [pg_ivm の公式リポジトリ](https://github.com/sraoss/pg_ivm) を参照してください。
