---
name: metricstream-record-loss-event
description: Record an operational loss event in MetricStream with its financial impacts and its risk/regulatory event-type mapping, for operational risk capital and Basel-style loss data collection.
api: metricstream-loss-event-management
operations:
  - createMsLsmInternalLossEvent
  - getMsLsmInternalLossEvent
  - patchMsLsmInternalLossEvent
  - createMsLsmImpacts
  - collectionsMsLsmImpacts
  - createMsLsmRskRegMap
  - collectionsMsLsmDefaultFields
generated: '2026-08-25'
method: generated
source: https://assets.metricstream.com/pdf/Developer-Portal/Loss_Event_Managment_API/ (MsLsmInternalLossEvent.html, MsLsmImpacts.html, MsLsmRskRegMap.html, MsLsmDefaultFields.html) and MslsmLSM%20API%20Overview.html (read 2026-08-25)
---

# Record an operational loss event

The Loss Event Management module is where operational losses are captured for operational-risk
reporting. Six resources, 36 operations.

## Steps

1. **Check the default currency configuration** so amounts are recorded in the unit the instance
   expects: `collectionsMsLsmDefaultFields` — `POST /collections/lsm/defaultcurrencyforlossevents`.
2. **Create the loss event.** `createMsLsmInternalLossEvent` — `POST /lsm/internallossevent`.
   (Use the parallel `createMsLsmExternalLossEvent` — `POST /lsm/externallossevent` — for
   externally-sourced consortium loss data.) Capture the returned `ObjectID`.
3. **Record the financial impacts.** `createMsLsmImpacts` — `POST /lsm/impact`, one per impact line.
4. **Map it to the risk / regulatory event type.** `createMsLsmRskRegMap` —
   `POST /lsm/riskeventtypemapping`. This is what makes the event countable in regulatory
   operational-risk categories.
5. **Verify.** `getMsLsmInternalLossEvent` — `GET /lsm/internallossevent/{id}` and
   `collectionsMsLsmImpacts` — `POST /collections/lsm/impact`.
6. **Amend.** `patchMsLsmInternalLossEvent` — `PATCH /lsm/internallossevent`.

## Rules an agent must respect

- **This is regulatory loss data.** A duplicated or mis-mapped loss event contaminates operational-risk
  capital and regulatory reporting. Treat every write as requiring human sign-off.
- **No idempotency, no reversal.** There is no idempotency key and no delete/void/reverse operation on
  any loss-event resource, and no correction window is documented. If you double-post an impact,
  the only remedy is a `patch` — and MetricStream does not publish whether the prior value is retained.
- **Amounts and currency.** Do not assume a currency; read the instance's configured default from
  step 1. The public docs do not publish the amount field's type or precision.
- **Approval rules exist server-side.** `MsLsmLossRule` (`/lsm/approvalrule`) configures approval
  routing. Do not modify approval rules from an agent; a loss event that skips approval is an audit
  finding.
- **Statuses only:** `200/400/401/403/404/422`. No 429, no 5xx, no problem+json.
