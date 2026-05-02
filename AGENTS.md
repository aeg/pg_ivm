# Repository Guidelines

## Project Structure & Module Organization
This repository packages `pg_ivm` into PostgreSQL Docker images. Version-specific files live under `16/`, `17/`, and `18/`. Each directory currently contains `init-db.sh`, which runs during container initialization and creates the `pg_ivm` extension. The root [`README.md`](/home/eiji/git/pg_ivm/README.md) documents image usage and example container startup commands.

## Build, Test, and Development Commands
Use Docker to build and validate changes.

- `docker build -t pg_ivm:local 17` builds the PostgreSQL 17 image from the `17/` directory.
- `docker build -t pg_ivm:local 16` builds the PostgreSQL 16 image from the `16/` directory.
- `docker run --rm -e POSTGRES_PASSWORD=postgres pg_ivm:local` starts a container and runs the init script.
- `docker exec -it <container> psql -U postgres -d postgres -c "\\dx"` verifies that `pg_ivm` is installed.

Run the same checks for every version directory you touch.

## Coding Style & Naming Conventions
Shell scripts should remain simple and portable:

- Use `#!/bin/bash` with `set -e`.
- Keep indentation consistent with existing scripts; current files use four spaces inside the SQL heredoc.
- Prefer clear, lowercase file names such as `init-db.sh`.
- Keep SQL initialization idempotent where possible and avoid adding unrelated setup logic.

## Testing Guidelines
There is no automated test suite in this repository. Validation is done by building the image and confirming that the extension is available in a fresh container. For any change to initialization logic, test the affected PostgreSQL version and confirm `CREATE EXTENSION pg_ivm;` succeeds without manual intervention.

## Commit & Pull Request Guidelines
Recent commit messages are short and release-oriented, for example: `support postgresql17beta3 + pg_ivm1.9` and `fix pg_ivm version`. Follow that style: concise, imperative or descriptive, and scoped to one change. Pull requests should include:

- the PostgreSQL and `pg_ivm` versions affected
- the exact build and verification commands used
- any image tag changes reflected in `README.md`

## Version Update Notes
When adding support for a new PostgreSQL release, create a matching top-level directory such as `18/`, copy the init script pattern, and keep behavior aligned across supported versions unless a version-specific difference is required.

## Agent Workflow
When updating PostgreSQL or `pg_ivm` versions in this repository, use the `pg-ivm-version-upgrade` skill from `~/.codex/skills/pg-ivm-version-upgrade`. It covers version checks, directory updates, `README.md` and `README.ja.md` updates, `docker build` validation, the JSON-based `pgivm.create_immv(...)` smoke test, and Docker Hub tag or push command preparation.
