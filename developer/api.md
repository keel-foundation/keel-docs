# API

**Status:** local demo contract. The v1 target surface is in RFC-0001 and `keel-spec/schemas/openapi.yaml`.

## Core

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness |
| `GET` | `/api/v1/channels/demo` | Demo channel |
| `GET` | `/api/v1/videos/demo` | Demo video |
| `GET` | `/api/v1/demo` | Demo video and channel |

## Moderation

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness |
| `GET` | `/v1/policies` | Active demo policy |
| `GET` | `/v1/demo/state` | Current state and ledger |
| `POST` | `/v1/demo/reset` | Clear action state, append a reset record |
| `POST` | `/v1/actions` | Create an enforcement action |
| `POST` | `/v1/actions/{action_id}/appeal` | File an appeal |
| `POST` | `/v1/appeals/{appeal_id}/decide` | Decide an appeal |
| `GET` | `/v1/ledger` | Ledger and verification |
| `GET` | `/v1/ledger/verify` | Verify the chain |
| `GET` | `/v1/ledger/export?format=jsonl` or `format=csv` | Export the ledger |

## Configuration

| Variable | Service | Default |
|---|---|---|
| `TUBEKEEL_CORE_ADDR` | core | `127.0.0.1:8080` |
| `TUBEKEEL_PIPELINE_ADDR` | pipeline | `127.0.0.1:8081` |
| `TUBEKEEL_CORS_ORIGINS` | core, pipeline, moderation | `http://localhost:3000` |
| `TUBEKEEL_LEDGER_PATH` | moderation | `data/ledger.jsonl` |
| `TUBEKEEL_RATE_LIMIT` | moderation | `30` mutating requests per IP per minute |
| `TUBEKEEL_DEMO_TOKEN` | moderation, web | unset |

## Errors

The moderation service returns JSON errors with a `detail` field. Common statuses:

- `401` — demo token required;
- `409` — an active action exists, the appeal window is closed, or the appeal is not allowed;
- `422` — request validation failed;
- `429` — rate limit exceeded.

The demo is local-only and not a production API.
