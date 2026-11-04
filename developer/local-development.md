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
- Go 1.23+ for core (API and media worker)
- Docker only if you are working on the private maintainer stack
- `gh` only for organization automation

## Run the public mock demo

The demo is scripted and mocked. It is not a production system and does not include a real
classifier, authentication, database, upload path, or production staff moderation portal.

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

cp .env.example .env
# edit KEELTUBE_DATABASE_URL in .env
go run ./cmd/seed
go run ./cmd/api
```

Core listens on `127.0.0.1:8080` by default. The seed is idempotent and creates the demo actor,
channel, and three video fixtures.

Database tests need a test database and serialized packages:

```sh
KEELTUBE_TEST_DATABASE_URL='postgres://USER:PASSWORD@HOST:PORT/DATABASE?sslmode=disable' \
  go test -p 1 ./...
go vet ./...
gofmt -l .
```

Without `KEELTUBE_TEST_DATABASE_URL`, the database tests skip. The CI workflow provides a
PostgreSQL service.

### Media worker

The media worker is part of `keel-core`. It requires an S3-compatible bucket and is separate from
the API process:

```sh
git clone https://github.com/keel-foundation/keel-core.git
cd keel-core
cp .env.example .env
# configure KEELTUBE_DATABASE_URL and KEELTUBE_MEDIA_S3_*
go run ./cmd/worker
go test ./...
go vet ./...
gofmt -l .
```

The worker exposes `/health` and `/status` on `127.0.0.1:8083` by default.

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

### Web (private, maintainers only)

`keel-web` is private until the maintainer pool is established. The commands below are for
maintainers with access.

```sh
git clone https://github.com/keel-foundation/keel-web.git
cd keel-web
npm ci
npm run dev
```

The web app runs at `http://localhost:3000`. The public feed, watch page, channel page, search,
library, creator studio, and creator moderation record use the local data adapter and need no
backend:

```sh
open http://localhost:3000
open http://localhost:3000/explore
open http://localhost:3000/watch/ksKCMmvOxwQ
open http://localhost:3000/channels/ch_maravale
open http://localhost:3000/studio
open http://localhost:3000/studio/videos
open http://localhost:3000/studio/moderation
```

The web app still reads through the local data adapter. Its Cloudflare Worker BFF is implemented
and proxies core and creator-safe moderation requests; the data adapters are the remaining step.
The creator studio and creator moderation record use a local workspace session; they are not
production access control. Staff moderation is the separate private `keel-moderation-web` portal
with its own privileged session.

Checks:

```sh
npm run lint
npm test
npm run typecheck
npm run build
npm run format:check
```

E2E starts the Vite dev server and uses the local data adapter. No backend stack is required:

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

The private dev stack includes PostgreSQL with pgvector, embeddings, the core seed job, core,
moderation, and an optional media worker:

```sh
docker compose -f keel-infra/devstack/apps.compose.yml up -d --build
# optional FFmpeg worker; requires an S3-compatible bucket in devstack/.env
docker compose -f keel-infra/devstack/apps.compose.yml --profile media up -d
```

Then run the web app on the host:

```sh
cd keel-web
npm run dev
```

Open `http://localhost:3000`. The web app still reads through the local data adapter; the backend
stack and Worker BFF are ready for the adapter phase.

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
- Core has PostgreSQL persistence and implements upload, transcoding/HLS in a separate worker,
  subscriptions, notifications, analytics, federation, and session authentication. Real classifier
  models, payout execution, and production hardening remain future work.
- The moderation workflow is persisted in PostgreSQL, with SQLite as an offline/test fallback. The
  audit ledger is append-only JSONL, not PostgreSQL plus an external anchor.
- The web studio and creator moderation record are fully built against the local data adapter. The
  Cloudflare Worker BFF is implemented, but the data adapters are not connected yet. Staff
  moderation is the separate private `keel-moderation-web` portal; the creator app does not expose
  moderator actions.
