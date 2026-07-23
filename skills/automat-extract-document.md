---
name: Extract structured data from a document
description: >-
  Use the Automat Extract API to pull structured JSON from a PDF or image against a configured
  extractor. Covers auth, the four file-input options, and error handling.
api: openapi/automat-extract-openapi.json
operations:
  - extract
---

# Extract structured data from a document (Automat)

Extract structured data from a document (PDF, image, or other supported format) using a
pre-configured Automat extractor. The extractor's schema determines the shape of the returned JSON.

## Prerequisites
- An Automat API key (created in the Automat dashboard / Studio).
- An `extractorId` for a configured extractor (from the dashboard or the API).

## Auth
All requests use bearer auth. Send the API key in the `Authorization` header:

```
Authorization: Bearer <api_key>
```

A missing, invalid, or revoked key returns `401 { "error": "unauthorized", "message": ... }`.

## Steps

1. **Choose a file-input method** for `extract` (`POST https://studio.runautomat.com/api/extract`,
   `multipart/form-data`). One of:
   - Binary file upload (max 4.5 MB)
   - Base64-encoded string (max 4.5 MB) — set `mimeType`
   - Data URL string (max 4.5 MB) — set `mimeType`
   - Public URL (up to 35 MB) — set `mimeType`

2. **Call `extract`** with the required fields `extractorId` and `file`. Optionally pin a version with
   `extractorVersionId`, override the model with `config`, or set a `timeout` (1–300 seconds).

   ```bash
   curl -X POST https://studio.runautomat.com/api/extract \
     -H "Authorization: Bearer $AUTOMAT_API_KEY" \
     -F "extractorId=$EXTRACTOR_ID" \
     -F "file=@invoice.pdf"
   ```

3. **Read the result.** On `200`, the `ExtractResponse.data` object holds the extracted fields
   matching your extractor's schema.

## Error handling
Errors use a custom envelope `{ "error": "<code>", "message": "<human readable>" }` (not RFC 9457):
- `400 bad_request` — missing/malformed fields; confirm `file` and `extractorId` are present.
- `401 unauthorized` — check the bearer API key.
- `422` — extractor ID unknown/inaccessible, unsupported format, or the document could not be processed.
- `500 internal_error` — retry, or contact support.

See `errors/automat-problem-types.yml` and `conventions/automat-conventions.yml`.
