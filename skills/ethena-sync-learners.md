---
name: Sync and manage Ethena learners
description: Enroll, read, update, and offboard learners in Ethena, paging through the roster with cursor pagination.
api: openapi/ethena-openapi-original.yml
operations: [getLearners, createLearner, getLearnerById, patchLearnerById, deleteLearnerById]
---

# Sync and manage Ethena learners

Use this skill to keep an external system (HRIS, IdP, data warehouse) in sync with
the Ethena learner roster.

## Authentication
Ethena uses HTTP Basic auth. Send `Authorization: Basic <base64(username:apiKey)>`.
The API key is provisioned by an Ethena representative; there is no self-serve key.
Base URL: `https://api.goethena.com`, all paths under `/v1/`.

## Steps
1. **List the roster** — call `getLearners`. Filter with repeatable query params
   `id`, `status`, `email`, `name`. Page with `limit` (default 25) and `cursor`.
   The response has `limit`, `hasMore`, and `data[]`. While `hasMore` is true,
   repeat with the next `cursor`.
2. **Enroll a new learner** — call `createLearner` with the required fields
   (`name`, `email`, `status`, `country`, `isManager`, `language`). `language`
   uses Unicode CLDR language formats.
3. **Read one learner** — call `getLearnerById` with the opaque `id`
   (pattern `^[a-zA-Z0-9]+$`, e.g. `abc123def4`).
4. **Update a learner** — call `patchLearnerById` with only the fields to change.
   A successful update returns `204 No Content`.
5. **Offboard a learner** — call `deleteLearnerById`; returns `204`.

## Error handling
Errors are RFC 7807 problem objects with a `title`, optional `detail`, and an
`invalid-params` array on validation errors. Handle `400` (validation),
`401` (bad credentials), and `404` (unknown id). See
`errors/ethena-problem-types.yml`.

## Notes
- The API does not document an idempotency key; guard retries on POST yourself.
- `trainingUrl` on a learner deep-links to their training experience.
