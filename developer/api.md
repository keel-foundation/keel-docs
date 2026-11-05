# API

**Status:** current UAT contract. Core is PostgreSQL-backed and feature-complete for UAT.
Moderation uses PostgreSQL workflow state (SQLite remains an offline/test fallback) plus an
append-only JSONL ledger. The v1 target surface is in RFC-0001 and
`keel-spec/schemas/openapi.yaml`.

## Core

The complete core contract is in `keel-spec/schemas/openapi.yaml`. The groups are:

- Catalog and discovery: feed, Explore, videos, related videos, shorts, search, categories,
  channels, and channel videos.
- Identity: register, email verification, login, logout, session, CSRF, and password reset.
- Viewer state: subscriptions, subscription feed, library, history, reactions, saves, comments,
  comment likes, and notifications.
- Creator Studio: channel, video CRUD, publish/unpublish, immutable versions, thumbnails, subtitles,
  uploads, scans, stats, and analytics.
- Media gateway: MP4, thumbnails, captions, subtitles, and HLS playback through short-lived
  presigned redirects.
- Notifications: `/api/v1/me/notifications`, unread count, read/read-all, and the optional
  `/internal/v1/notifications` service ingest route.
- Creator moderation gateway: `/api/v1/studio/moderation/summary`, cases, actions, appeals,
  ledger, ledger verification/export, and appeal filing. Core authenticates the viewer session and
  calls `keel-moderation` with a server-side creator credential; the browser never receives a
  moderation role token.
- Reports and trust: `/api/v1/reports`, `/api/v1/me/reports`, `/api/v1/reports/{id}`, and
  `/api/v1/videos/{id}/trust`.
- Playlists and collections: `/api/v1/me/playlists` and `/api/v1/playlists/{id}`.
- Billing and settlement: `/api/v1/studio/billing/*` for Stripe Connect onboarding and
  `/internal/v1/billing/settlements` for idempotent settlements. `/webhooks/stripe` verifies
  `account.updated` and `transfer.*` events.
- Federation: WebFinger, ActivityPub actor/outbox/inbox documents, PeerTube-compatible video
  objects, a signed policy, and internal operator routes under `/internal/v1/federation`.

`GET /api/v1/channels/{id}` also accepts a handle such as `@maravale` or a bare handle.
`GET /api/v1/studio/videos/{id}` returns the edit aggregate: video fields plus `rights`,
`subtitles`, and `versions`.

Demo fixtures remain available at `/api/v1/channels/demo`, `/api/v1/videos/demo`, and
`/api/v1/demo`.

## Moderation

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Liveness |
| `GET` | `/v1/policies` | Active demo policy |
| `GET` | `/v1/demo/state` | Current state and ledger |
| `POST` | `/v1/demo/reset` | Clear workflow state, append a reset record |
| `GET` | `/v1/whoami` | Current principal and role |
| `POST` | `/v1/signals` | Ingest a viewer report and open a case (service token) |
| `GET` | `/v1/public/videos/{video_id}/record` | Public creator-safe case/action/appeal record |
| `POST` | `/v1/actions` | Create an enforcement action (moderator) |
| `POST` | `/v1/actions/{action_id}/appeal` | File an appeal (creator) |
| `POST` | `/v1/appeals/{appeal_id}/decide` | Decide an appeal (appeals reviewer) |
| `GET` | `/v1/appeals?status=pending` or `status=decided` | List appeals (moderator or appeals) |
| `GET` | `/v1/appeals/{appeal_id}` | Read an appeal (moderator or appeals) |
| `GET` | `/v1/summary` | Staff queue counts and ledger status |
| `GET` | `/v1/cases` | List cases for staff review (`status`, `channel_id`, `video_id`, paging) |
| `GET` | `/v1/cases/{case_id}` | Read one case |
| `POST` | `/v1/cases/{case_id}/actions` | Record enforcement for a case (moderator) |
| `POST` | `/v1/cases/{case_id}/dismiss` | Dismiss a case without enforcement (moderator) |
| `GET` | `/v1/actions` | List enforcement actions |
| `GET` | `/v1/actions/{action_id}` | Read one enforcement action |
| `GET` | `/v1/creator/*` | Creator-scoped summary, cases, actions, appeals, and ledger |
| `GET` | `/v1/ledger` | Ledger and verification |
| `GET` | `/v1/ledger/verify` | Verify the chain |
| `GET` | `/v1/ledger/export?format=jsonl` or `format=csv` | Export the ledger |

### Roles

The service supports `creator`, `moderator`, `appeals`, and `operator` roles. The preferred
production mode is OIDC: set `KEELTUBE_OIDC_ISSUER` and either `KEELTUBE_OIDC_JWKS_URL` or
`KEELTUBE_OIDC_HMAC_SECRET`. The principal ID, role, and channel scope come from the `sub`, `role`,
and `channel_ids` claims. Core mints short-lived HS256 claims for the creator moderation gateway.

Role-token mode sets `KEELTUBE_CREATOR_TOKEN`, `KEELTUBE_MODERATOR_TOKEN`, and
`KEELTUBE_APPEALS_TOKEN` to three distinct values. If neither OIDC nor role tokens are configured,
authenticated endpoints fail closed with `503`.

The appeals reviewer must differ from the original decision-maker and from the appellant. The
moderation service enforces this; the web client must not be the only guard.

Creator reads use `/v1/creator/*` and are channel-scoped from the authenticated principal, never
from a client-supplied channel. The local demo still leaves `/v1/demo/state` and the top-level
ledger read endpoints unauthenticated so the mock demo can load. This is not a production access
model. A shared deployment must add read authorization and redaction before exposing those
endpoints.

## Configuration

| Variable | Service | Default |
|---|---|---|
| `KEELTUBE_CORE_ADDR` | core | `127.0.0.1:8080` |
| `KEELTUBE_DATABASE_URL` | core | required; set in `.env` |
| `KEELTUBE_DEV_ACTOR_ID` | core | `usr_demo` |
| `KEELTUBE_DEV_ACTOR_TOKEN` | core | unset |
| `KEELTUBE_DEMO_MEDIA_URL` | core seed | web demo sample |
| `KEELTUBE_WORKER_ADDR` | core worker | `127.0.0.1:8083` |
| `KEELTUBE_WEB_PUBLIC_URL` | core seed | `http://localhost:3000` |
| `KEELTUBE_CORS_ORIGINS` | core, moderation | `http://localhost:3000` |
| `KEELTUBE_LEDGER_PATH` | moderation | `data/ledger.jsonl` |
| `KEELTUBE_MODERATION_DATABASE_URL` | moderation | unset; PostgreSQL workflow state (preferred) |
| `KEELTUBE_DATABASE_URL` | moderation | unset; PostgreSQL fallback |
| `KEELTUBE_STATE_DB` | moderation | `data/state.sqlite3`; SQLite fallback |
| `KEELTUBE_RATE_LIMIT` | moderation | `30` mutating requests per IP per minute |
| `KEELTUBE_SIGNAL_INGEST_TOKEN` | moderation | unset; enables `POST /v1/signals` |
| `KEELTUBE_CLASSIFIER_BACKEND` | moderation | `auto`; bundled linear model |
| `KEELTUBE_CLASSIFIER_MODEL` | moderation | unset; Hugging Face model for the `transformers` backend |
| `KEELTUBE_CLASSIFIER_URL` | moderation | unset; optional external classifier |
| `KEELTUBE_CORE_URL` | moderation | unset; creator notification delivery |
| `KEELTUBE_CORE_INTERNAL_TOKEN` | moderation, core | unset; internal notification ingest |
| `KEELTUBE_STRIPE_SECRET_KEY` | core | unset; enables Stripe Connect billing |
| `KEELTUBE_STRIPE_WEBHOOK_SECRET` | core | unset; enables the Stripe webhook route |
| `KEELTUBE_STRIPE_CONNECT_COUNTRY` | core | `US` |
| `KEELTUBE_STRIPE_CURRENCY` | core | `usd` |
| `KEELTUBE_STRIPE_MIN_SETTLEMENT_CENTS` | core | `100` |
| `KEELTUBE_MODERATION_URL` | core | unset |
| `KEELTUBE_MODERATION_CLAIMS_SECRET` | core | unset; 32+ characters enables OIDC-style claims |
| `KEELTUBE_MODERATION_CLAIMS_ISSUER` | core | `keel-core` |
| `KEELTUBE_MODERATION_CLAIMS_AUDIENCE` | core | `keel-moderation` |
| `KEELTUBE_OIDC_ISSUER` | moderation | unset; enables OIDC mode |
| `KEELTUBE_OIDC_AUDIENCE` | moderation | unset |
| `KEELTUBE_OIDC_JWKS_URL` | moderation | issuer discovery |
| `KEELTUBE_OIDC_HMAC_SECRET` | moderation | unset |
| `KEELTUBE_OIDC_ALGORITHMS` | moderation | `RS256,ES256` or `HS256` |
| `KEELTUBE_CREATOR_TOKEN` | moderation | unset |
| `KEELTUBE_MODERATOR_TOKEN` | moderation | unset |
| `KEELTUBE_APPEALS_TOKEN` | moderation | unset |

## Errors

The moderation service returns JSON errors with a `detail` field. Common statuses:

- `401` — a valid bearer token is required;
- `403` — the role is not allowed, or the reviewer is not independent;
- `404` — the action or appeal does not exist;
- `409` — an active action exists, the appeal window is closed, or the appeal is not allowed;
- `422` — request validation failed;
- `429` — rate limit exceeded.

The demo is local-only and not a production API.
