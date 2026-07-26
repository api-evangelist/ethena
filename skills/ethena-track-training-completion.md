---
name: Track Ethena training completion
description: Report on training campaigns and learner progress, and subscribe to completion webhooks to drive downstream workflows.
api: openapi/ethena-openapi-original.yml
operations: [getTrainingCampaigns, getTrainingCampaignById, getLearnerTrainingCampaigns, getLearnerTrainingCampaignById, getLearnerTrainingModules, createWebhook, getWebhookById]
---

# Track Ethena training completion

Use this skill to build compliance reporting and event-driven automation (for
example, gating system access on training status).

## Authentication
HTTP Basic auth: `Authorization: Basic <base64(username:apiKey)>`.
Base URL `https://api.goethena.com`, paths under `/v1/`. Webhooks are opt-in and
must be enabled by an Ethena representative — webhook operations return `403`
until the feature is on.

## Reporting steps
1. **List campaigns** — `getTrainingCampaigns` (includes a campaign `status`
   field); read one with `getTrainingCampaignById`.
2. **Learner progress per campaign** — `getLearnerTrainingCampaigns`, filtered by
   `learnerStatus` and `trainingCampaignStatus`; read one with
   `getLearnerTrainingCampaignById`.
3. **Module-level progress** — `getLearnerTrainingModules`, filtered by
   `learnerStatus` and `trainingCampaignStatus`.
   All list endpoints use cursor pagination (`limit`/`cursor` ->
   `limit`/`hasMore`/`data`).

## Event-driven steps
4. **Subscribe** — `createWebhook` with your HTTPS URL for the
   `learnerTrainingCampaignCompleted` event. The create response does NOT return
   the secret.
5. **Fetch the secret** — `getWebhookById` returns the per-webhook secret key.
6. **Verify each delivery** — compute HMAC SHA-256 (hex) of the raw request body
   using the secret and compare, in constant time, with the `X-Signature`
   header. The payload contains `learnerId`, `trainingCampaignId`, and
   `statusTimestamp`. See `asyncapi/ethena-webhooks.yml`.

## Error handling
RFC 7807 problem objects; expect `401` (auth), `403` (webhooks not enabled),
`404` (unknown id). See `errors/ethena-problem-types.yml` and
`conventions/ethena-conventions.yml`.
