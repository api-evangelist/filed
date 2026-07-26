---
name: filed-onboard-client
description: Create a new tax client in a Filed workspace and ingest its source documents into the binder.
api: Filed GraphQL API
endpoint: https://router.apps.filed.com/graphql
operations:
  - exchangeSurfaceRefreshTokenForAccessTokens
  - me
  - get_file_upload_info
  - createClient
  - addClientDocuments
  - getClientBinderMissingItems
generated: '2026-07-19'
method: generated
source: https://docs.apps.filed.com/guides/recipes/onboard-a-client
---

# Onboard a client and ingest documents

Create a new client (taxpayer/return) under a Filed workspace and load its source
documents so the binder can begin ingestion.

## Prerequisites
- A workspace-scoped **read_write** API key (Plugins → Filed API in the web app).
- All GraphQL calls POST to `https://router.apps.filed.com/graphql`.

## Steps

1. **Authenticate.** Call `exchangeSurfaceRefreshTokenForAccessTokens` (public, no
   token) with your API key as `refreshToken`. Keep the returned `workspaceToken`;
   send it as `Authorization: Bearer <workspaceToken>` on every later call. It
   expires in ~30 minutes — re-exchange on `UNAUTHENTICATED`.
2. **Confirm the workspace.** Run `me { ... on WorkspaceUser { workspace { id name } } }`
   to verify the token is scoped to the right firm.
3. **Stage each document.** Call the MCP `get_file_upload_info` tool (or the
   documented resumable upload endpoint) to get a short-lived (10-min) upload
   token + tus URL, drive the resumable upload against that URL, and collect the
   resulting `uploadId` for each file.
4. **Create the client.** Call `createClient(input: CreateClientInput!)` with
   `name`, `externalId`, `returnType`, `taxYear`, and the `uploadIds[]` from step 3.
   This optionally kicks off binder ingestion and returns `{ client, taskId }`.
5. **Add more documents later** (existing client) with `addClientDocuments`,
   passing the client id and new `uploadIds`.
6. **Track completeness.** Poll `getClientBinderMissingItems` to see the
   missing-item checklist and whether ingestion is done.

## Conventions & errors
- IDs are opaque UUIDv7 strings — round-trip them verbatim.
- List fields use `offset`/`limit`; there is no cursor or totalCount.
- Failures come back in the GraphQL `errors[]` array with `extensions.code`
  (`GRAPHQL_VALIDATION_FAILED` for a bad query). See errors/filed-problem-types.yml.
- There is **no idempotency key**; do not blindly retry `createClient` — check via
  `me { workspace { clients } }` first if a call's outcome is uncertain.
