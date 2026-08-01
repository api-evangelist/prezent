---
name: Convert a deck to a brand template
description: Convert an existing presentation into a brand-compliant Prezent template and download the result.
api: openapi/prezent-openapi-original.yml
operations: [createTemplateConversion, getTemplateConverterStatusV2, getTemplateConversionDownload]
---

# Convert a deck to a brand template

Use the Prezent Template Converter to reformat an existing deck to an approved brand template.

## Auth
Send `Authorization: Bearer <api_key>`. See `authentication/prezent-authentication.yml`.

## Steps
1. **Pick a template** — `GET /api/v1/templates` (`listTemplates`) to choose a `template_id`.
2. **Start the conversion** — `POST /api/v1/template-conversions` (`createTemplateConversion`)
   with the source deck and `template_id`. Include an `Idempotency-Key` header. The response
   returns a `callback_id`.
3. **Poll (v2, strict)** — `GET /api/v2/template-converter/status/{callback_id}`
   (`getTemplateConverterStatusV2`): HTTP 200 only on success, 4xx/5xx on workflow failure.
   Or subscribe to `template_conversion.completed` / `template_conversion.failed` webhooks.
4. **(optional) Review** — `GET /api/v1/template-conversions/{callback_id}/review-suggestions`
   (`listTemplateConversionReviewSuggestions`) and apply changes via
   `updateTemplateConversionReviewSuggestions` / `createTemplateConversionTemplateChange`.
5. **Download** — `GET /api/v1/template-conversions/{callback_id}/download`
   (`getTemplateConversionDownload`).

## Rules
- Uniform envelope + stable `error.code` catalog (`errors/prezent-error-codes.yml`).
- Prefer the REST-resource paths; legacy `/api/v1/template-converter/*` paths return a
  `Deprecation` header (`lifecycle/prezent-lifecycle.yml`).
- Honor rate-limit headers (`conventions/prezent-conventions.yml`).
