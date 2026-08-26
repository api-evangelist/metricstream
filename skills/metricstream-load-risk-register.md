---
name: metricstream-load-risk-register
description: Load and maintain risks, controls and evidence in the MetricStream GRC Foundation object model, including bulk loads from an upstream risk register.
api: metricstream-grc-foundation
operations:
  - createMsGrcRisk
  - creationsMsGrcRisk
  - getMsGrcRisk
  - patchMsGrcRisk
  - updatesMsGrcRisk
  - collectionsMsGrcRisk
  - createMsGrcControl
  - collectionsMsGrcControl
  - createMsGrcEvidence
  - collectionsMsGrcEvidence
generated: '2026-08-25'
method: generated
source: https://assets.metricstream.com/pdf/Developer-Portal/GRCF_API/MsGrcRisk.html, MsGrcControl.html, MsGrcEvidence.html and https://assets.metricstream.com/pdf/Developer-Portal/GRCF_API/MsGrcGRC%20API%20Overview.html (read 2026-08-25)
---

# Load a risk register into MetricStream GRC Foundation

GRC Foundation is the shared object model the rest of the MetricStream platform references —
Risk, Control, Evidence, Requirement, Standard, Regulatory Body, Asset, Process and eleven more,
each with the same six-operation surface.

## The uniform surface

Every GRC Foundation entity `{entity}` exposes exactly these, and nothing else:

| Purpose | operationId pattern | Method + path |
|---|---|---|
| Read one | `getMsGrc{Entity}` | `GET /grc/{entity}/{id}` |
| Create one | `createMsGrc{Entity}` | `POST /grc/{entity}` |
| Update one | `patchMsGrc{Entity}` | `PATCH /grc/{entity}` |
| Read many by ID | `collectionsMsGrc{Entity}` | `POST /collections/grc/{entity}` |
| Create many | `creationsMsGrc{Entity}` | `POST /bulk/grc/{entity}` |
| Update many | `updatesMsGrc{Entity}` | `PATCH /bulk/grc/{entity}` |

Learn it once and it holds for all 18 GRC Foundation entities (108 operations).

## Steps

1. **Reconcile before you write.** `collectionsMsGrcRisk` — `POST /collections/grc/risk` with the
   `ObjectID`s you already hold. This is the only safe way to avoid duplicates; there is no
   idempotency key.
2. **Create the risks.** `createMsGrcRisk` — `POST /grc/risk` one at a time, or
   `creationsMsGrcRisk` — `POST /bulk/grc/risk` for a batch.
3. **Create or reconcile the controls.** `createMsGrcControl` — `POST /grc/control`,
   `collectionsMsGrcControl` — `POST /collections/grc/control`.
4. **Attach evidence.** `createMsGrcEvidence` — `POST /grc/evidence`.
5. **Maintain.** `patchMsGrcRisk` — `PATCH /grc/risk` for a single record,
   `updatesMsGrcRisk` — `PATCH /bulk/grc/risk` for a batch.

## Body shape

MetricStream models each entity as a header object plus named form SECTIONS and GROUPFIELDS —
`Details`, `OwnershipAndSecurity`, `AdditionalDetails`, `Comments`, and groupfields such as `Dates`,
`Approvers`, `Hierarchy`, `Validity`, `Classification`, `Attachments`, `Relationships`. An App Studio
form maps onto the JSON body one-for-one. The exact field names for YOUR instance come from the
reference page for that entity — MetricStream publishes no downloadable spec, and the sections are
configurable per deployment, so read them from the instance rather than hard-coding.

## Rules an agent must respect

- **Read before write, always.** No idempotency key exists on any of the 204 operations.
- **No delete, no undo.** There is no delete, restore or rollback operation on any GRC Foundation
  entity, and no retention window is published. A `patchMsGrcRisk` that overwrites a risk rating
  cannot be reversed through the API. Require explicit human approval for every write.
- **Bulk is the sharpest edge.** `creationsMsGrcRisk` and `updatesMsGrcRisk` write many audit records
  in one call with no documented partial-failure semantics, no published maximum batch size, and no
  reversal. Never let an agent issue a bulk write unattended.
- **Statuses only.** `200/400/401/403/404/422`. `403` means a missing platform activity, not a bad
  credential. No 429 and no 5xx are declared.
