---
name: veritus-drip-campaign
description: Enroll a customer into an omnichannel (voice/SMS/email) Veritus drip campaign and manage its lifecycle.
generated: '2026-07-21'
method: generated
source: https://docs.veritus.com/guides/core-concepts/drip-campaigns
api: Veritus API
base_url: https://app.veritus.com/api/v1
operations:
  - POST /customers
  - POST /customers/{customerId}/drip-campaigns
  - DELETE /customers/{customerId}/drip-campaigns/{dripCampaignId}
---

# Run a Veritus omnichannel drip campaign

Enroll a customer into an automated, compliance-gated sequence that orchestrates
voice calls, voicemail, SMS, and email until the customer engages.

## Steps

1. **Create or reuse the customer** — `POST /customers` (see externalId dedup /
   409 behavior). Keep the returned `id`.
2. **Enroll in the campaign** — `POST /customers/{customerId}/drip-campaigns`
   with `campaignName` (the campaign is pre-configured with your Veritus rep) and
   an optional `webhook.url` (+ `webhook.secretId` for signing). The customer runs
   through the compliance engine before the first step is scheduled.
3. **Track progress via webhooks** — you receive `drip-campaign.call-created`,
   `drip-campaign.call-ended`, and `drip-campaign.sms` events, each with the
   campaign `status` (`in-progress` / `completed` / `canceled`). Verify the
   `X-Webhook-Signature`.
4. **Auto-cancellation** — the campaign cancels itself automatically the moment
   the customer replies or answers, to prevent over-contact.
5. **Cancel manually (optional)** — `DELETE /customers/{customerId}/drip-campaigns/{dripCampaignId}`
   with a `reason` of `customer-converted` or `customer-engaged`.

## Rules
- Campaign sequences and compliance parameters are configured with a Veritus
  representative; the API enrolls and cancels, it does not define steps.
- Honor compliance 422s and log `requestId`.
- See conventions/veritus-conventions.yml and asyncapi/veritus-webhooks-asyncapi.yml.
