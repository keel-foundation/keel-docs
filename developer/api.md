# API

**Status:** local demo contract. Core is PostgreSQL-backed; moderation remains SQLite plus JSONL.
The v1 target surface is in RFC-0001 and `keel-spec/schemas/openapi.yaml`.

## Core

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness and database ping |
| `GET` | `/api/v1/channels/demo` | Demo channel |
| `GET` | `/api/v1/channels/{id}` | Channel by id |
| `GET` | `/api/v1/videos/demo` | Demo video |
| `GET` | `/api/v1/videos/{id}` | Video by id |
| `GET` | `/api/v1/demo` | Demo video and channel |
| `GET` | `/api/v1/me` | Current dev principal |

## Moderation

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness |
| `GET` | `/v1/policies` | Active demo policy |
| `GET` | `/v1/demo/state` | Current state and ledger |
| `POST` | `/v1/demo/reset` | Clear workflow state, append a reset record |
| `GET` | `/v1/whoami` | Current principal and role |
| `POST` | `/v1/actions` | Create an enforcement action (moderator) |
| `POST` | `/v1/actions/{action_id}/appeal` | File an appeal (creator) |
| `POST` | `/v1/appeals/{appeal_id}/decide` | Decide an appeal (appeals reviewer) |
| `GET` | `/v1/appeals?status=pending` or `status=decided` | List appeals (moderator or appeals) |
| `GET` | `/v1/appeals/{appeal_id}` | Read an appeal (moderator or appeals) |
| `GET` | `/v1/ledger` | Ledger and verification |
| `GET` | `/v1/ledger/verify` | Verify the chain |
| `GET` | `/v1/ledger/export?format=jsonl` or `format=csv` | Export the ledger |

### Roles

The service supports `creator`, `moderator`, and `appeals` roles. Set
`KEELTUBE_CREATOR_TOKEN`, `KEELTUBE_MODERATOR_TOKEN`, and `KEELTUBE_APPEALS_TOKEN` to three
distinct values to enforce them. If they are unset, the service runs in open local mode. The
legacy `KEELTUBE_DEMO_TOKEN` simulates all three roles with one token.

The appeals reviewer must differ from the original decision-maker and from the appellant. The
moderation service enforces this; the web client must not be the only guard.

The local demo leaves `/v1/demo/state` and the ledger read endpoints unauthenticated so the mock
demo can load. This is not a production access model. A shared deployment must add read
authorization, per-creator scoping, and redaction before exposing those endpoints.

## Configuration

| Variable | Service | Default |
|---|---|---|
| `KEELTUBE_CORE_ADDR` | core | `127.0.0.1:8080` |
| `KEELTUBE_DATABASE_URL` | core | required; set in `.env` |
| `KEELTUBE_DEV_ACTOR_ID` | core | `usr_demo` |
| `KEELTUBE_DEV_ACTOR_TOKEN` | core | unset |
| `KEELTUBE_PIPELINE_ADDR` | pipeline | `127.0.0.1:8081` |
| `KEELTUBE_PIPELINE_PUBLIC_URL` | core seed | `http://localhost:8081` |
| `KEELTUBE_WEB_PUBLIC_URL` | core seed | `http://localhost:3000` |
| `KEELTUBE_CORS_ORIGINS` | core, pipeline, moderation | `http://localhost:3000` |
| `KEELTUBE_LEDGER_PATH` | moderation | `data/ledger.jsonl` |
| `KEELTUBE_STATE_DB` | moderation | `data/state.sqlite3` |
| `KEELTUBE_RATE_LIMIT` | moderation | `30` mutating requests per IP per minute |
| `KEELTUBE_DEMO_TOKEN` | moderation, web | unset |
| `KEELTUBE_CREATOR_TOKEN` | moderation | unset |
| `KEELTUBE_MODERATOR_TOKEN` | moderation | unset |
| `KEELTUBE_APPEALS_TOKEN` | moderation | unset |

## Errors

The moderation service returns JSON errors with a `detail` field. Common statuses:

- `401` — a role or demo token is required;
- `403` — the role is not allowed, or the reviewer is not independent;
- `404` — the action or appeal does not exist;
- `409` — an active action exists, the appeal window is closed, or the appeal is not allowed;
- `422` — request validation failed;
- `429` — rate limit exceeded.

The demo is local-only and not a production API.
