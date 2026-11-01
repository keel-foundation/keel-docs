# Local development

**Last reviewed:** 1 November 2026

This page covers running the **public** KeelTube modules and the **mock demo**. It does not
describe the private production stack.

There is no all-in-one public `make` target for the whole ecosystem. Use the command for the
module you are changing. The public `keel-dev` Makefile provides `make demo`, `make demo-open`,
`make test`, `make check`, and `make boundary`.

## Prerequisites

- Node.js 22+
- Python 3.9+
- Go 1.23+ for core and pipeline
- Docker only if you are working on the private maintainer stack
- `gh` only for organization automation

## Run the public mock demo

The demo is scripted and mocked. It is not a production system and does not include a real
classifier, authentication, database, upload path, or production moderation console.

```sh
git clone https://github.com/keel-foundation/keel-dev.git
cd keel-dev
make demo
```

Open `http://localhost:3000`. Use `make demo-open` to open the browser automatically.

Run the demo checks with:

```sh
make test
```

## Run one module

You do not need to clone the ecosystem. Clone only the repository you are changing.

### Core

Core needs PostgreSQL. Point `KEELTUBE_DATABASE_URL` at a local database, seed it, then run the
API:

```sh
git clone https://github.com/keel-foundation/keel-core.git
cd keel-core

export KEELTUBE_DATABASE_URL='postgres://keeltube:keeltube-dev@127.0.0.1:5432/keeltube?sslmode=disable'
go run ./cmd/seed
go run ./cmd/api
```

Core listens on `127.0.0.1:8080` by default. The seed is idempotent and creates the demo actor,
channel, and three video fixtures.

Database tests need a test database and serialized packages:

```sh
KEELTUBE_TEST_DATABASE_URL='postgres://keeltube:keeltube-dev@127.0.0.1:5432/keeltube?sslmode=disable' \
  go test -p 1 ./...
go vet ./...
gofmt -l .
```

Without `KEELTUBE_TEST_DATABASE_URL`, the database tests skip. The CI workflow provides a
PostgreSQL service.

### Pipeline

```sh
git clone https://github.com/keel-foundation/keel-pipeline.git
cd keel-pipeline
go run ./cmd/worker
go test ./...
go vet ./...
gofmt -l .
```

Pipeline listens on `127.0.0.1:8081` by default.

### Moderation

```sh
git clone https://github.com/keel-foundation/keel-moderation.git
cd keel-moderation
python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.lock
.venv/bin/pip install -e . --no-deps
.venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8082
```

In another terminal:

```sh
.venv/bin/pytest -q
.venv/bin/ruff check app tests
```

The contract tests validate responses against `keel-spec`. Clone `keel-spec` as a sibling of
`keel-moderation` (`../keel-spec`) to run them; otherwise those tests skip.

### Web

```sh
git clone https://github.com/keel-foundation/keel-web.git
cd keel-web
npm ci
npm run dev -- -H 127.0.0.1
```

The web app runs at `http://localhost:3000`. The landing page and `/demo` work without extra
services; `/watch/[id]` needs core with the demo seed:

```sh
open http://localhost:3000/watch/vid_demo_001
```

Checks:

```sh
npm test
npm run typecheck
npm run build
npm run format:check
```

Watch-page E2E (requires the backend stack and a Playwright browser):

```sh
npx playwright install chromium
npm run test:e2e
```

### Spec

```sh
git clone https://github.com/keel-foundation/keel-spec.git
cd keel-spec
python3 -m venv .venv
.venv/bin/pip install -r requirements-dev.lock
.venv/bin/python scripts/validate_schemas.py
```

## Point the mock demo at one local service

The public demo can proxy to local services while you work:

```sh
cd keel-dev
KEELTUBE_MODE=remote \
KEELTUBE_API_BASE_URL=http://127.0.0.1:8080 \
KEELTUBE_MODERATION_BASE_URL=http://127.0.0.1:8082 \
make demo
```

Use stage endpoints for services you are not changing. The demo server keeps the token
server-side. See [stage mode](https://github.com/keel-foundation/keel-dev/blob/main/docs/staging.md)
in the `keel-dev` repository.

## Backend in Docker (maintainers only)

The private dev stack includes PostgreSQL, the core seed job, core, pipeline, and moderation:

```sh
docker compose -f keel-infra/devstack/apps.compose.yml up -d --build
```

Then run the web app on the host:

```sh
cd keel-web
npm ci
npm run dev -- -H 127.0.0.1
```

Open `http://localhost:3000/watch/vid_demo_001`.

## Full-stack development (maintainers only)

Full-stack local setup is reserved for the author and maintainer group. It lives in the private
`keel-infra` repository:

- guide: `keel-infra/maintainers/local-full-stack.md`
- bootstrap: `keel-infra/maintainers/scripts/bootstrap-dev.sh --all`
- dev stack: `docker compose -f keel-infra/devstack/docker-compose.yml up -d`

Do not clone or copy private repositories into a public workspace, and do not push private
material to a public repository. If private repositories are present in the same workspace, run
`make boundary` from `keel-dev` before pushing.

## Limitations

- The public `keel-dev` demo is mocked: the signal is synthetic and the flow is scripted.
- Core has PostgreSQL persistence for actors, channels, and videos, but upload, transcoding,
  subscriptions, and production auth are not implemented.
- The watch page plays the demo clip with MP4 fallback; HLS is not implemented yet.
- The moderation workflow is persisted in SQLite; the audit ledger is append-only JSONL, not
  PostgreSQL plus an external anchor.
- The `/demo` page is a demonstration, not a production moderation console.
