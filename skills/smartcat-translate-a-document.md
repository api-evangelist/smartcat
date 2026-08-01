---
name: Translate a document with Smartcat
description: Create a project, upload a source document, wait for translation, and export the result.
api: openapi/smartcat-openapi.json
operations:
  - POST /api/integration/v1/project/create
  - POST /api/integration/v1/project/document
  - GET /api/integration/v1/document
  - POST /api/integration/v1/document/export
  - GET /api/integration/v1/document/export/{taskId}
---

# Translate a document with Smartcat

Use the Smartcat integration API (`/api/integration/v1`) to run a source file
through translation and get the translated file back.

## Auth
HTTP Basic on every call: `Authorization: Basic base64("<AccountId>:<ApiKey>")`.
Create credentials under **Settings > API**. Pick the regional host that matches
your account: `smartcat.com` (EU), `us.smartcat.com` (Americas), `ea.smartcat.com` (Asia).
Stay under **4 requests/second**.

## Steps
1. **Create a project** — `POST /api/integration/v1/project/create` with the project
   name, source language, and target language(s). Capture the returned `projectId`.
2. **Add the document** — `POST /api/integration/v1/project/document` (multipart) to
   upload the source file into the project. Capture the returned `documentId`.
3. **Poll status** — `GET /api/integration/v1/document?documentId=...` until the
   document status shows translation is complete.
4. **Request export** — `POST /api/integration/v1/document/export` with the document
   id(s). Capture the returned export `taskId`.
5. **Download** — `GET /api/integration/v1/document/export/{taskId}` to fetch the
   translated file once the export task is ready.

## Error & retry rules
- Errors come back as `ProblemDetails` (`type`/`title`/`status`/`detail`) or an
  `ErrorResponse` carrying a `requestId` — log the `requestId` for support.
- `401` means bad Basic credentials; `402` means the account balance/plan blocks the
  action; `404` means a bad project/document id.
- No idempotency-key header exists — do not blindly retry `POST`s; check state first.
- On `429`/rate issues, back off to stay within 4 req/s.
