# Moderation ledger

The audit ledger is the first trust primitive in the local moderation service.

## Record shape

```json
{
  "seq": 1,
  "type": "enforcement_action.created",
  "at": "2026-10-31T00:00:00+00:00",
  "payload": {
    "action": {
      "id": "act_demo0001",
      "case_id": "case_demo0001"
    },
    "actor": {
      "id": "demo-reviewer",
      "role": "moderator"
    }
  },
  "prev_hash": "GENESIS",
  "hash": "sha256:..."
}
```

The hash is:

```
SHA-256(canonical_json({seq, type, at, payload, prev_hash}))
```

`prev_hash` and `seq` are part of the hash input. Reordering, relinking, or editing any record
breaks verification. The `actor` block records who caused the record; the domain object is kept
under a named key (`action`, `case`, or `appeal`) so the actor does not alter its schema.

## Properties

- Append-only JSONL file.
- Verified on load and through `GET /v1/ledger/verify`.
- Resetting demo state appends a `demo.reset` record; it never truncates the ledger.
- Export as JSONL or CSV through `GET /v1/ledger/export`.
- The demo stores the ledger locally; it is not externally anchored.
- Workflow state (cases, actions, appeals) is stored in PostgreSQL with versioned migrations;
  SQLite is an offline/test fallback. The ledger remains JSONL.

## Roles and actors

The service supports three roles:

- `creator` files appeals.
- `moderator` creates enforcement actions and can list the queue.
- `appeals` decides appeals and must be independent of the original decision-maker.

In OIDC mode, the role and channel scope come from verified claims. In role-token mode, each role
uses a separate bearer token with fixed local principals. If neither mode is configured, the service
fails closed.

## What is still missing

- A PostgreSQL-backed append-only ledger.
- A signed periodic anchor outside the database.
- Redaction for public transparency aggregates.
- A transparency report generated from the ledger.

Creator record reads are channel-scoped through `/v1/creator/*`; the top-level demo read endpoints
remain unauthenticated for the local demo. Until the anchor and transparency work land, the ledger
demonstrates the contract, not production tamper resistance.
