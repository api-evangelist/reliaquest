---
generated: '2026-09-18'
method: generated
name: Work digital-risk-protection alerts
description: List GreyMatter Digital Risk Protection (DRP) alerts, take ownership, comment, move them through states, and bulk-close resolved ones.
api: postman/reliaquest-greymatter-operations.txt
operations: [drpAlerts, drpAlert, assignDRPAlert, watchDRPAlert, addDrpAlertComment, updateDrpAlertState, bulkCloseDrpAlerts]
source: >-
  Grounded in ReliaQuest's public GreyMatter API Postman collection (https://apidocs.myreliaquest.com/).
  Every operation name verified verbatim in the collection; auth per authentication/reliaquest-authentication.yml,
  conventions per conventions/reliaquest-conventions.yml, errors per errors/reliaquest-problem-types.yml.
---

# Work digital-risk-protection alerts

DRP alerts are the external-exposure findings (impersonation, leaked credentials, dark-web mentions) that GreyMatter Digital Risk Protection raises. They are addressed by Global ID and by a human-readable `shortCode`.

## Auth and transport
- POST to `https://greymatter.myreliaquest.com/graphql` with `X-API-KEY`. See `authentication/reliaquest-authentication.yml`.

## Steps
1. **List alerts** — `drpAlerts` with Relay pagination (`first`, `after`). Each `edges[].node` carries `id`, `shortCode`, `currentState`, `currentAssignment`, `assetMatches`, `takedown` and `activityLogItems`.
2. **Read one** — `drpAlert(id)` for the full record before acting.
3. **Take ownership** — `assignDRPAlert(input: DRPAlertAssignmentInput!)` with `shortCode`, `assignee` (user Global ID) and an optional `comment`; the response's `unsuccessfulDrpAlerts` lists any that could not be assigned. `watchDRPAlert` subscribes you without assigning.
4. **Comment** — `addDrpAlertComment` (edit with `updateDrpAlertComment`, remove with `deleteDrpAlertComment`).
5. **Move it** — `updateDrpAlertState(input: UpdateDrpAlertStateInput!)` with `drpAlert` (id), `state` (e.g. `IN_PROGRESS`) and `comment`.
6. **Close in bulk** — `bulkCloseDrpAlerts(input: BulkCloseDrpAlertsInput!)` with `drpAlertIds[]` and a `comment`; read `updated[]` and `failed[]` — a partial failure is returned, not raised.

## Undo
- Assignment and watch have explicit reversals (`unassignDRPAlert`, `unWatchDRPAlert`); state can be set again with `updateDrpAlertState`. No window is published.

## Rules
- Bulk mutations return per-item `failed[]` — always inspect it; `success` alone does not mean every id was updated.
- No idempotency key; re-read before retrying a comment or state change after a timeout.
