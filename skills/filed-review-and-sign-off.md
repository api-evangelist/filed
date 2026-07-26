---
name: filed-review-and-sign-off
description: Read a client's leadsheets and review items in Filed, then record reviewer sign-offs and annotations.
api: Filed GraphQL API
endpoint: https://router.apps.filed.com/graphql
operations:
  - getClientBinderSubdocuments
  - getClientDocumentMessages
  - createDocumentMessage
  - createDocumentMessageThread
  - updateDocumentMessage
generated: '2026-07-19'
method: generated
source: https://docs.apps.filed.com/guides/recipes/review-and-sign-off
---

# Review a return and sign off

Read a client's leadsheets and review items, annotate binder documents, and record
reviewer sign-offs through the document-message write surface.

## Prerequisites
- A workspace-scoped **read_write** API key and a fresh `workspaceToken`.
- A `clientId` whose tax prep run has produced review items.

## Steps

1. **Read the binder.** Query `getClientBinderSubdocuments` to list the client's
   documents/forms and drill into field-level trace and sourcing on leadsheets.
2. **Read existing annotations.** Query `getClientDocumentMessages` for notes,
   flags, and prior reviewer sign-offs on a document.
3. **Annotate.** Add a note or flag with `createDocumentMessage`; thread a reply
   with `createDocumentMessageThread`; edit with `updateDocumentMessage`. Hide or
   restore a message with `hideDocumentMessage` / `unhideDocumentMessage`.
4. **Sign off.** Record a reviewer sign-off via the document-message write surface
   (a sign-off message on the relevant document/leadsheet item) to mark the item
   reviewed.

## Conventions & errors
- Documents and messages are reached through the client's binder under
  `me { ... on WorkspaceUser { workspace { clients } } }`.
- IDs are opaque UUIDv7 strings; round-trip verbatim.
- A read_only key can read messages but every write mutation is rejected.
- Failures come back in the GraphQL `errors[]` array. See errors/filed-problem-types.yml.
