---
name: metricstream-report-issue
description: Report a control weakness, deviation or anomaly into the MetricStream Issues module and follow it up, using the published Business REST API.
api: metricstream-issues
operations:
  - createMsIsmIssue
  - getMsIsmIssue
  - patchMsIsmIssue
  - creationsMsIsmIssue
  - collectionsMsIsmIssue
generated: '2026-08-25'
method: generated
source: https://assets.metricstream.com/pdf/Developer-Portal/Issues_API/MsIsmIssue.html and https://assets.metricstream.com/pdf/Developer-Portal/MsIsmISM-API-Overview.html (read 2026-08-25)
---

# Report an issue into MetricStream

MetricStream's Issues module lets first-line users flag a weakness or gap in internal controls, or
a deficiency in an internal process. This skill drives that from an upstream system.

## Before you start

- **Base URL is per-customer.** The runtime form is
  `https://{METRICSTREAM_INSTANCE}/metricstream/b2b/api/{EXTERNAL_API_VERSION}/{RESOURCE}` —
  for example `https://your-instance.example.com/metricstream/b2b/api/7.0/ism/issue`.
  There is no shared public host. Get the instance hostname from the MetricStream administrator.
- **Version rule.** Client and server must share the same MAJOR version. The client may request a
  LOWER minor than the server, never a higher one.
- **Authentication scheme is not published.** Every operation declares
  `401 - Required Authentication information is missing or invalid`, and the docs say alternate
  authentication schemes exist, but the header/credential form is only available from MetricStream
  Support. Do not guess it. See `authentication/metricstream-authentication.yml`.
- **Authorization is activity-based.** A `403 Forbidden - Insufficient user privileges` means the
  calling user lacks the platform ACTIVITY for that endpoint — it is not a bad credential. Escalate
  to the MetricStream administrator, do not retry.

## Steps

1. **Create the issue.**
   `createMsIsmIssue` — `POST /ism/issue`. Success is `200` with the issue object (not `201`).
   Capture the returned `ObjectID`; it is the only handle you get.
2. **Read it back to confirm.**
   `getMsIsmIssue` — `GET /ism/issue/{id}`, where `{id}` is that `ObjectID` (declared type String).
3. **Update it as the investigation progresses.**
   `patchMsIsmIssue` — `PATCH /ism/issue`.
4. **Loading many at once.**
   `creationsMsIsmIssue` — `POST /bulk/ism/issue` for a batch create, and
   `collectionsMsIsmIssue` — `POST /collections/ism/issue` to fetch a set by IDs.

## Rules an agent must respect

- **There is no idempotency key.** If `createMsIsmIssue` times out you cannot safely retry it —
  MetricStream publishes no replay-safety guarantee and no request-id echo. Read back with
  `collectionsMsIsmIssue` before creating again, or you will duplicate an audit record.
- **There is no reversal.** The published surface has no delete, cancel, void or restore operation
  on Issues, and no retention or undo window is documented. Treat every create and patch as final
  and require human confirmation before writing.
- **Bulk has no documented partial-failure envelope.** `creationsMsIsmIssue` may accept a batch, but
  MetricStream does not publish what happens when one member fails. Prefer single creates until the
  behaviour is confirmed with the vendor.
- **Errors are bare HTTP statuses.** The declared set is `200, 400, 401, 403, 404, 422` — no
  `application/problem+json`, no error-code registry, no 429, no 5xx. Do not parse an error body
  shape; branch on the status. See `errors/metricstream-problem-types.yml`.
- **No rate limits are published.** No `RateLimit-*` header, no `Retry-After`, and 429 is not
  declared. Self-throttle conservatively.
