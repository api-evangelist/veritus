---
name: veritus-import-analyze-interaction
description: Import historical SMS/email interactions into Veritus and run idempotent outcome analysis on them.
generated: '2026-07-21'
method: generated
source: https://docs.veritus.com/llms.txt
api: Veritus API
base_url: https://app.veritus.com/api/v1
operations:
  - POST /customers/{customerId}/interactions/sms/import
  - POST /customers/{customerId}/interactions/email/import
  - POST /interactions/{interactionId}/analyze
---

# Import and analyze customer interactions

Sync historical conversations from your systems into Veritus and extract outcome
variables (e.g. stop-requests, human-follow-up-requested).

## Steps

1. **Import an SMS** — `POST /customers/{customerId}/interactions/sms/import`
   with `direction`, `timestamp` (ISO-8601), `from`/`to` (E.164), and `body`.
   For many rows use `.../sms/import/bulk` (up to 100, partial success).
2. **Import an email** — `POST /customers/{customerId}/interactions/email/import`
   with `direction`, `timestamp`, `from`, `to[]`, `subject`, and at least one of
   `plainTextBody` / `htmlBody`. Bulk variant at `.../email/import/bulk`.
3. **Capture the interaction id** — each import returns `data.id` and `threadId`.
4. **Analyze** — `POST /interactions/{interactionId}/analyze` (optionally
   `applyOutcomes:true`) to extract `outputVariables`. This call is **idempotent**:
   repeating it returns the existing analysis rather than re-running.

## Rules
- Bulk endpoints return `200` with per-item `results[]` — inspect each item's
  `success`/`error`, do not assume all rows succeeded.
- Analysis output fields depend on your organization's configuration.
- See conventions/veritus-conventions.yml (idempotency, bulk) and errors/veritus-problem-types.yml.
