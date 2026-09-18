---
generated: '2026-09-18'
method: generated
name: Run a response playbook and monitor the run
description: Discover which GreyMatter playbooks are recommended for an incident's artifacts, run one against the customer's connected integrations, poll the run, and rerun failed tasks.
api: postman/reliaquest-greymatter-operations.txt
operations: [recommendedPlaybooks, playbooks, customerPlaybooks, runPlaybook, playbookRun, playbookRuns, rerunAllFailedTasksForPlaybookRun]
source: >-
  Grounded in ReliaQuest's public GreyMatter API Postman collection (https://apidocs.myreliaquest.com/).
  Every operation name verified verbatim in the collection; auth per authentication/reliaquest-authentication.yml,
  conventions per conventions/reliaquest-conventions.yml, errors per errors/reliaquest-problem-types.yml.
---

# Run a response playbook and monitor the run

Playbooks execute response actions (containment, enrichment, ticketing) in the customer's connected technologies. **This is the irreversible part of the API** — a run cannot be undone (see `conventions/reliaquest-conventions.yml`, reversibility), so confirm intent before step 3.

## Auth and transport
- POST to `https://greymatter.myreliaquest.com/graphql` with `X-API-KEY`. See `authentication/reliaquest-authentication.yml`.

## Steps
1. **See what is available** — `playbooks` (ReliaQuest-provided; each has `action`, `fieldDefinitions`, `supportedTechnologies`) and `customerPlaybooks` (customer-authored; `supportedIntegrations`). Note `rqAllowedToRun` where returned.
2. **Ask for recommendations for a specific incident** — `recommendedPlaybooks(filter: RecommendedPlaybookFilter!)` with the incident's `artifacts { field, values }` (from `incident.allArtifacts`) and, where relevant, `tempRuleId`. The response lists each playbook with the `fields` it requires (`field`, `required`, `values`).
3. **Run it** — `runPlaybook(input: RunPlaybookInput!)`. Supply the playbook, the `originatorId` (the incident/alert Global ID), `input.integrationIds` (which connected technologies to act through — from `integrations`) and `input.playbookVariables { field, values }` matching the required fields from step 2. Read back `success` and `playbookRun.id`.
4. **Poll the run** — `playbookRun(id)` for state and `executionResult`; `playbookRuns` lists runs with filters and Relay pagination.
5. **Recover failures** — `rerunAllFailedTasksForPlaybookRun(id)` re-executes only the failed tasks of a run.

## Rules
- No idempotency key exists. **Never retry `runPlaybook` on a timeout without first checking `playbookRuns` for a run with your `originatorId`** — a duplicate run executes the response action twice.
- Each Node entity returned counts against the 5,000-token/hour budget; poll `playbookRun` with a narrow selection.
