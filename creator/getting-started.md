# Try the local appeal demo

**Last reviewed:** 1 November 2026

## Prerequisites

- Python 3.9+

The public mock demo has no backend, database, Node.js, or Go requirement.

## Run

```sh
git clone https://github.com/keel-foundation/keel-dev.git
cd keel-dev
make demo
```

Open `http://localhost:3000`. Use `make demo-open` to open the browser automatically.

## What you will see

1. A synthetic automated signal flags an original documentary as mass-produced spam.
2. A case and enforcement action are created with a Statement of Reasons.
3. You file an appeal with grounds and a statement.
4. An independent appeals reviewer reverses the action with written reasoning.
5. The audit ledger shows the chain, verification result, and JSONL export.

The whole flow runs in the browser tab. Reloading the page starts again from the initial state.

## What this is not

This is a mocked, scripted demo, not a production service. The classifier is synthetic, the
reviewer is simulated, and the ledger is browser-local. There is no real authentication,
database, classifier, or independent review. Do not expose the demo to the public internet.

For the optional Next.js demo that calls the local moderation service, see the `keel-web`
README and the `keel-moderation` README.
