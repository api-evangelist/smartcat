---
name: Receive Smartcat status callbacks
description: Register an account webhook URL and monitor delivery so you get project and document status changes.
api: openapi/smartcat-openapi.json
operations:
  - POST /api/integration/v1/callback
  - GET /api/integration/v1/callback
  - GET /api/integration/v1/callback/lastErrors
  - DELETE /api/integration/v1/callback
---

# Receive Smartcat status callbacks

Smartcat pushes status changes to a single account-level callback URL instead of an
AsyncAPI event stream.

## Auth
HTTP Basic (`<AccountId>:<ApiKey>`), same regional hosts as the rest of the API.

## Steps
1. **Register the callback** — `POST /api/integration/v1/callback` with your public
   HTTPS endpoint. Smartcat will POST to `<callbackURL>/project/status` and
   `<callbackURL>/document/status`.
2. **Verify registration** — `GET /api/integration/v1/callback` to read back the
   current notification settings.
3. **Handle events** — accept `project.status` and `document.status` POSTs at your
   endpoint and respond `2xx` quickly; do the work asynchronously.
4. **Monitor failures** — `GET /api/integration/v1/callback/lastErrors` to see recent
   failed deliveries. Smartcat retries with exponential backoff (~30s initial, up to
   8 attempts), so make your handler idempotent on the incoming id.
5. **Remove** — `DELETE /api/integration/v1/callback` to stop notifications.

## Notes
- Only one callback URL per account; route by the `/project/status` vs
  `/document/status` path suffix.
- Errors surface as `ProblemDetails`/`ErrorResponse` with a `requestId`.
