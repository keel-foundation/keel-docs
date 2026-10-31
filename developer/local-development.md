# Local development

**Last reviewed:** 31 October 2026

## Prerequisites

- Node.js 22+
- Python 3.9+
- Go 1.23+ for core and pipeline (optional for the demo)
- Docker for the local PostgreSQL/LocalStack S3/NATS stack
- `gh` only for organization automation

## Workspace

```sh
git clone https://github.com/keel-foundation/keel-dev.git
cd keel-dev
./scripts/bootstrap-dev.sh
```

The bootstrap clones the public repos and the `.github` repo. Use `--all` to clone private repos you can access.

## Services

```sh
make infra-up       # optional; not used by the demo yet
make core-dev
make pipeline-dev
make moderation-dev
make web-dev
```

`make moderation-dev` uses the project venv when present.

## Checks

```sh
make test
make test-demo
make validate-schemas
make check
```

## Demo

```sh
make demo
```

The demo starts moderation and web. If Go is installed it also starts core and pipeline; if either fails, the UI falls back to the bundled sample clip. Services bind to localhost and CORS is restricted to the web origin.

## Limitations

- No production auth, database, queue, or object storage integration yet.
- The classifier signal is synthetic.
- The ledger is append-only JSONL, not PostgreSQL plus an external anchor.
