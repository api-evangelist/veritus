---
name: veritus-outbound-call
description: Place a compliance-checked outbound voice call to a customer with the Veritus Agent API and retrieve the recording.
generated: '2026-07-21'
method: generated
source: https://docs.veritus.com/guides/core-concepts/creating-calls
api: Veritus API
base_url: https://app.veritus.com/api/v1
operations:
  - POST /customers
  - POST /customers/{customerId}/calls
  - POST /calls/get-signed-url
---

# Place an outbound Veritus voice call

Use this skill to create a customer and place a single compliance-checked
outbound voice call, then fetch the recording.

## Prerequisites
- A bearer API key (`Authorization: Bearer sk-...`), issued by Veritus per environment.
- An `agentId` assigned to you by Veritus.
- Base URL: `https://app.veritus.com/api/v1` (production) or `https://sandbox.veritus.com/api/v1` (sandbox).

## Steps

1. **Create the customer** — `POST /customers` with `externalId`, name, `email`,
   `primaryPhone` (E.164), and `address`. For collections customers also send
   `ssnLastFour`, `dateOfBirth`, and a `collections` object with `totalBalance`.
   A `409` means a customer with that `externalId` already exists (treat as
   idempotent and reuse the id).
2. **Place the call** — `POST /customers/{customerId}/calls` with the assigned
   `agentId` and an optional `webhook.url` (+ `webhook.secretId` to sign).
   The customer runs through the compliance engine first.
3. **Handle compliance** — a `200` returns `data.compliance` with
   `checksPassed`/`checksFailed`. A `422` means the call was blocked; read
   `publicFacingMessage` for the failed rule (e.g. respectful-hours, frequency
   limit) and retry when the condition is met.
4. **Receive the result** — the `call-ended` webhook delivers `endedReason`,
   `recordingUrl` (signed, ~30-min TTL), `durationMs`, transcript `messages[]`,
   and `outputVariables`. Verify the `X-Webhook-Signature` header first.
5. **Re-fetch the recording** — if the signed URL expired,
   `POST /calls/get-signed-url` with the `callId` for a fresh 30-minute link.

## Rules
- Every response carries a `requestId`; log it for support.
- Verify webhook signatures: HMAC SHA-256 over `{timestamp}.{raw_body}`, reject
  timestamps older than 5 minutes.
- See conventions/veritus-conventions.yml and errors/veritus-problem-types.yml.
