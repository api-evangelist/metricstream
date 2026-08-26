---
name: metricstream-post-kri-metrics
description: Define a key risk / key performance indicator in MetricStream and feed it periodic data points from an upstream monitoring or data-warehouse system.
api: metricstream-metrics
operations:
  - createMsMetDefinition
  - getMsMetDefinition
  - patchMsMetDefinition
  - collectionsMsMetDefinition
  - createMsMetDataEntry
  - creationsMsMetDataEntry
  - collectionsMsMetDataEntry
generated: '2026-08-25'
method: generated
source: https://assets.metricstream.com/pdf/Developer-Portal/Metric_API/MsMetDefinition.html, MsMetDataEntry.html and MsMetMET%20API%20Overview.html (read 2026-08-25)
---

# Feed KRI/KPI data into MetricStream

Two resources: the metric DEFINITION (`/met/metric`) and the metric DATA (`/met/metricdata`).
This is the highest-frequency integration on the MetricStream API surface — a scheduled feed rather
than a one-off write — so the throttling and idempotency gaps below matter most here.

## Steps

1. **Find or create the definition.**
   `collectionsMsMetDefinition` — `POST /collections/met/metric` to look up by known `ObjectID`s;
   `createMsMetDefinition` — `POST /met/metric` to create; `getMsMetDefinition` —
   `GET /met/metric/{id}` to read one back.
2. **Post the data points.**
   `createMsMetDataEntry` — `POST /met/metricdata` for a single reading, or
   `creationsMsMetDataEntry` — `POST /bulk/met/metricdata` for a period batch.
3. **Verify what landed.**
   `collectionsMsMetDataEntry` — `POST /collections/met/metricdata`.
4. **Correct a definition.** `patchMsMetDefinition` — `PATCH /met/metric`.

## Rules an agent must respect

- **Verify-then-write on every cycle.** With no idempotency key, a retried batch double-counts a KRI.
  Always `collectionsMsMetDataEntry` for the period before re-posting it.
- **No published rate limit.** No `RateLimit-*` header, no `Retry-After`, and 429 is not declared on
  any operation. For a scheduled high-frequency feed, pick a conservative fixed rate and a fixed
  batch size, and agree both with the MetricStream administrator rather than discovering the ceiling
  in production.
- **No published maximum batch size** for `creationsMsMetDataEntry`, and no partial-failure envelope.
- **No delete.** A wrong data point cannot be removed through the published API. Confirm the value
  before posting.
- **Pagination is `limit`/`offset`**, documented as a platform-wide convention — but neither the
  default nor the maximum page size is published.
