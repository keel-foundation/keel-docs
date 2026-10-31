# Moderation ledger

The audit ledger is the first trust primitive in the local demo.

## Record shape

```json
{
  "seq": 1,
  "type": "enforcement_action.created",
  "at": "2026-10-31T00:00:00+00:00",
  "payload": {},
  "prev_hash": "GENESIS",
  "hash": "sha256:..."
}
```

The hash is:

```
SHA-256(canonical_json({seq, type, at, payload, prev_hash}))
```

`prev_hash` and `seq` are part of the hash input. Reordering, relinking, or editing any record breaks verification.

## Properties

- Append-only JSONL file.
- Verified on load and through `GET /v1/ledger/verify`.
- Resetting demo state appends a `demo.reset` record; it never truncates the ledger.
- Export as JSONL or CSV through `GET /v1/ledger/export`.
- The demo stores the ledger locally; it is not externally anchored.

## What is still missing

- A PostgreSQL-backed append-only ledger.
- A signed periodic anchor outside the database.
- Per-creator access control over records.
- Redaction for public aggregates.
- A transparency report generated from the ledger.

Until those exist, the demo demonstrates the contract, not production tamper resistance.
