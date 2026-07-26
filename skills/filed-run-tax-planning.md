---
name: filed-run-tax-planning
description: Start a Filed tax advisor run for a client, read the resulting plan, and update a strategy's status.
api: Filed GraphQL API
endpoint: https://router.apps.filed.com/graphql
operations:
  - triggerTaxAdvisor
  - pollTaxAdvisor
  - clientAdvisorPlan
  - taxAdvisorResult
  - setAdvisorStrategyStatus
generated: '2026-07-19'
method: generated
source: https://docs.apps.filed.com/guides/recipes/run-tax-planning
---

# Run tax planning / advisory

Start a tax advisor run for a client, poll it to completion, read the plan and its
strategies, and update a strategy's status.

## Prerequisites
- A workspace-scoped **read_write** API key and a fresh `workspaceToken`.
- A `clientId` whose return and source documents are already ingested.

## Steps

1. **Start the advisor run.** Call the `triggerTaxAdvisor` mutation (the docs also
   show `InitiateTaxAdvisor`) for the client. It returns a `runId` / `taskId` for a
   background advisor task.
2. **Poll to completion.** Use `pollTaxAdvisor` (or `taskResult`) until the task
   reports `COMPLETED`.
3. **Read the plan.** Query `clientAdvisorPlan` / `taxAdvisorResult` to get the
   `AdvisorPlan`: `globalSummary`, `strategies[]` (`AdvisorStrategy`), and the
   JSON rollups `byDomain`, `bySavingsHorizon`, `estimatedSavingsCentsByHorizon`.
   Each `AdvisorStrategy` carries `strategyId`, `domain`, `title`, `summary`,
   `applicabilityEvidence`, `implementationPlan[]`, `estimatedSavingsCents`,
   `savingsHorizon`, and `status`.
4. **Update a strategy.** Call `setAdvisorStrategyStatus` with the `runId` and the
   strategy id to accept/dismiss a recommendation.

## Conventions & errors
- `byDomain` and the other `JSON` scalars return the whole value with no
  sub-selection — parse them client-side and keep the parser lenient.
- IDs are opaque UUIDv7 strings; round-trip verbatim.
- Failures come back in the GraphQL `errors[]` array. See errors/filed-problem-types.yml.
