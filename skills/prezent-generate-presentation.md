---
name: Generate a Prezent presentation
description: Turn a text prompt plus optional source files into an on-brand presentation and download it.
api: openapi/prezent-openapi-original.yml
operations: [createAutogeneration, getAutogeneration, createAutogenerationDownload]
---

# Generate a Prezent presentation

Use the Prezent Platform API (base `https://api.prezent.ai`) to build a brand-compliant
presentation from a prompt and optional uploaded files.

## Auth
Send `Authorization: Bearer <api_key>` on every call. Keys are scoped to specific paths; an
out-of-scope call returns `404 ENDPOINT_NOT_FOUND`. The `x-api-key` header is not accepted.

## Steps
1. **(optional) Upload sources** — `POST /api/v1/upload` for any source files, then
   `POST /api/v1/preprocess` / `POST /api/v1/validate` (operationIds `uploadFile`,
   `preprocessFile`, `validateFiles`).
2. **Start the build** — `POST /api/v1/autogenerations` (`createAutogeneration`) with the prompt,
   plus any `audience_id`/`theme_id`. Include an `Idempotency-Key` header (a UUID) so a retry never
   creates two jobs. The response returns a `callback_id`.
3. **Poll** — `GET /api/v1/autogenerations/{callback_id}` (`getAutogeneration`) until complete.
   Alternatively subscribe to the `autogeneration.completed` / `autogeneration.failed` webhooks.
4. **Download** — `POST /api/v1/autogenerations/{callback_id}/downloads`
   (`createAutogenerationDownload`) to get the finished file.

## Rules
- Responses use the uniform envelope `{ "success": true, "data": {...} }` (or `success:false` with
  an `error.code` from the catalog in `errors/prezent-error-codes.yml`).
- Respect `X-RateLimit-*` / `Retry-After` headers; back off on `429 TOO_MANY_REQUESTS` /
  `RATE_LIMIT_EXCEEDED` / `USAGE_LIMIT_EXCEEDED`.
- See `conventions/prezent-conventions.yml` for idempotency, pagination, and async details.
