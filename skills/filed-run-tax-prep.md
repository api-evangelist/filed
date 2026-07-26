---
name: filed-run-tax-prep
description: Start a Filed tax prep run for a client and read the review items it produces.
api: Filed GraphQL API
endpoint: https://router.apps.filed.com/graphql
operations:
  - triggerTaxPrep
  - listTasks
  - taskResult
  - clientTaskStatus
generated: '2026-07-19'
method: generated
source: https://docs.apps.filed.com/guides/recipes/run-tax-prep
---

# Run tax prep end to end

Kick off a tax prep run for a client and follow the background task to completion
to read its Review Action List.

## Prerequisites
- A workspace-scoped **read_write** API key and a fresh `workspaceToken`
  (see filed-onboard-client, step 1).
- A `clientId` with source documents already in its binder.

## Steps

1. **Start the run.** Call the `triggerTaxPrep(input: TriggerTaxPrepInput!)`
   mutation with `clientId` and `returnType` (optional: `software`,
   `softwareClientId`, `runDataEntry`, `forceReconcile`). It returns the `taskId`
   of a `TAX_PREP` background task.
2. **Poll to completion.** Query the task with `taskResult` (or `clientTaskStatus`
   / `listTasks(type: TAX_PREP)`) until `status: COMPLETED`. Back off between polls;
   there is no webhook.
3. **Read results.** When complete, the task's `result` resolves to
   `TaskTaxPrepResult`: `summary`, `documentCount`, `extractedFormCount`,
   `reviewItemCount`, and `reviewItems[]` (each a `TaxPrepReviewItem` with
   `severity`, `category`, `description`).
4. **Act on review items.** Surface high-severity items for a preparer.
   Back-office operations `retriggerTaxPrepStep` and `setTaxPrepTaskStatus` require
   a **user** token, not the workspaceToken — only use them if you hold one.

## Conventions & errors
- Tasks are reached through `me { ... on WorkspaceUser { workspace { tasks } } }`
  or per-client `tasks(...)`; you never pass a workspace id.
- Poll, don't push: long-running AI runs complete asynchronously.
- Errors return in the GraphQL `errors[]` array; a read_only key will have its
  `triggerTaxPrep` mutation rejected. See errors/filed-problem-types.yml.
