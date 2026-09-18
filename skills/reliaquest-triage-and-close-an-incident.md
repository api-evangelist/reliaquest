---
generated: '2026-09-18'
method: generated
name: Triage and close a GreyMatter incident
description: List open GreyMatter incidents, acknowledge and assign one, add investigation notes, and close it with a close code — optionally registering a detection exclusion for a confirmed false positive.
api: postman/reliaquest-greymatter-operations.txt
operations: [incidents, incident, acknowledgeIncident, assignIncident, addIncidentComment, updateIncidentState, closeIncident, unresolveIncident]
source: >-
  Grounded in ReliaQuest's public GreyMatter API Postman collection (https://apidocs.myreliaquest.com/).
  Every operation name verified verbatim in the collection; auth per authentication/reliaquest-authentication.yml,
  conventions per conventions/reliaquest-conventions.yml, errors per errors/reliaquest-problem-types.yml.
---

# Triage and close a GreyMatter incident

Move an incident from "new" to "closed" through the GreyMatter GraphQL API.

## Auth and transport
- POST every request to `https://greymatter.myreliaquest.com/graphql` with a JSON body `{"query": ..., "variables": ...}` and the header `X-API-KEY: <key>` (minted in GreyMatter > Settings > API Key Management; keys expire, default one year). Plain HTTP and unauthenticated calls fail (403).
- To work incidents for a non-default company entity, add `x-reliaquest-customer: <headerSlug>` (get the slug from the `customers` query).
- Budget: 5,000 tokens per user per hour, where every Node entity returned counts as a token. Keep `first` small and select only the fields you need; call `rateLimit { cost }` to check spend.

## Steps
1. **Find candidates** — `incidents(after, first, incidentFilter, incidentOrder)`. Filter with `IncidentFilter` (e.g. `acknowledged: false`, a `closed`/`created` date range, `assignees`), page with `first`/`after`, and read `edges[].node.id` (a Global ID) plus `category`, `detectedAt`, `escalatedAt`, `originator`. Stop paging when `pageInfo.hasNextPage` is false.
2. **Read the incident** — `incident(id)`. Pull `description`, `allArtifacts`, `rule`, `comments`, `activity` and `assignee` to decide what to do.
3. **Acknowledge it** — `acknowledgeIncident(input: IncidentAcknowledgementInput!)` with `incidentId`, an `acknowledgementMethod`, and `autoAssign: true` to take it yourself.
4. **Assign it (if not auto-assigned)** — `assignIncident(input: AssignIncidentInput!)` with `incidentId`, `assigneeId` (a user Global ID from `user`/`me`) and an optional `comment`.
5. **Record findings** — `addIncidentComment` as you investigate; `updateIncidentState(input: UpdateIncidentStateInput!)` changes state and adds a comment in one call.
6. **Close it** — `closeIncident(request: CloseIncidentRequest)` with `incidentId`, a `closeCode` and `closeNote`. Read `success`, `incident.closedAt` and `incident.closeCode` back.
   - For a confirmed false positive, set `createExclusion: true`, `closeCode: CUSTOMER_FALSE_POSITIVE`, `excludeArtifacts { field, values }` and optionally `exclusionExpiresAt`. Any other close code with `createExclusion: true` is rejected as a validation error. The incident closes first; if the exclusion rule cannot be created the mutation still returns, with `exclusionFailureReason` populated and `exclusionRule` null — check it.

## Undo
- `unresolveIncident` reopens a resolved incident. No time window is published; assume none is guaranteed.

## Rules
- There is no idempotency key: a retried `closeIncident` is harmless, but do not retry `addIncidentComment` or `updateIncidentState` blindly after a timeout — re-read the incident first.
- Errors before the GraphQL layer come back as `{timestamp, status, error, path, errors:[{detail}]}` (403 = bad/missing key); GraphQL errors come back in `errors[]`. See `errors/reliaquest-problem-types.yml`.
