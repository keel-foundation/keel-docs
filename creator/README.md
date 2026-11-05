# Creator documentation

## Current status

KeelTube is in UAT preparation. Core is PostgreSQL-backed and implements identity, catalog,
engagement, discovery, creator uploads/transcoding/HLS, subscriptions, notifications, analytics,
viewer reports and trust records, and ActivityPub federation. Moderation has durable PostgreSQL
workflow state with a role-separated creator appeal flow and an append-only audit ledger. The
private `keel-moderation-web` staff portal implements the staff review, enforcement, appeal,
actions, ledger, and federation surfaces.

The public artifact is still the local appeal demo in `keel-dev`. The fully built web client in
`keel-web` (feed, watch, channels, search, library, creator studio, and creator moderation record)
is private and maintainer-only while the maintainer pool is established. Its UI is complete on
local data and its Cloudflare Worker BFF is implemented; the data adapters are the remaining UAT
integration step. Stripe Connect onboarding and manual settlement are implemented in core; viewer
billing and content-protection fingerprinting are not implemented yet.

## Guides

- [Creator rights](../governance/creator-rights.md)
- [Appeals](../governance/appeals.md)
- [Local demo](getting-started.md)
- [Export](export.md)

## Principles

- Your archive and audience relationship are yours.
- Every enforcement action has a specific reason, evidence, a human decision, and an appeal route.
- Export is available before, during, and after an enforcement action.
- Piracy, impersonation, spam, scams, and fraud are treated as serious harms.

This documentation will expand as the alpha ships. Every page should state what it covers and when it was last reviewed.
