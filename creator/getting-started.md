# Try the local appeal demo

**Last reviewed:** 31 October 2026

## Prerequisites

- Node.js 22+
- Python 3.9+
- `curl`
- `nc`
- Go 1.23+ is optional; without it the UI uses the bundled sample clip.

## Run

```sh
git clone https://github.com/keel-foundation/keel-dev.git
cd keel-dev
./scripts/bootstrap-dev.sh
make demo
```

Open the URL printed by the script, usually `http://localhost:3000/demo`.

## What you will see

1. A synthetic automated signal flags an original documentary as mass-produced spam.
2. A case and enforcement action are created with a Statement of Reasons.
3. You file an appeal with grounds and a statement.
4. An independent appeals reviewer reverses the action with written reasoning.
5. The audit ledger shows the chain, verification result, and JSONL export.

## What this is not

This is a local demo, not a production service. The classifier is synthetic, authentication is optional, and the ledger is a local JSONL file. Do not expose the demo to the public internet.
