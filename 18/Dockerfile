# build stage
FROM postgres:18.4 AS builder

RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    git \
    postgresql-server-dev-18

RUN git clone --branch v1.15 --depth 1 https://github.com/sraoss/pg_ivm.git /usr/src/pg_ivm

RUN cd /usr/src/pg_ivm && \
    make USE_PGXS=1 && \
    make USE_PGXS=1 install

# runtime stage
FROM postgres:18.4

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN mkdir -p '/usr/lib/postgresql/18/lib' && \
    mkdir -p '/usr/share/postgresql/18/extension' && \
    mkdir -p '/usr/lib/postgresql/18/lib/bitcode/pg_ivm'

COPY --from=builder /usr/lib/postgresql/18/lib/pg_ivm.so /usr/lib/postgresql/18/lib/pg_ivm.so
COPY --from=builder /usr/share/postgresql/18/extension/pg_ivm* /usr/share/postgresql/18/extension/
COPY --from=builder /usr/lib/postgresql/18/lib/bitcode/pg_ivm/* /usr/lib/postgresql/18/lib/bitcode/pg_ivm/
COPY --from=builder /usr/lib/postgresql/18/lib/bitcode/pg_ivm.index.bc /usr/lib/postgresql/18/lib/bitcode/pg_ivm.index.bc

COPY init-db.sh /docker-entrypoint-initdb.d/
