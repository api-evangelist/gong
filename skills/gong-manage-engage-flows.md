---
name: Manage Gong Engage flows and prospects
description: List Engage flows and folders, assign CRM prospects to a flow, check what they are already assigned to, and unassign them — including the two different key spaces Gong uses for unassignment.
api: openapi/gong-flows-api-openapi.yml, openapi/gong-folders-api-openapi.yml, openapi/gong-prospects-api-openapi.yml
operations: [listEngageFlows, listFlowFolders, listAssignedFlowsForProspects, assignProspectsToFlow, unassignProspectsFromFlow, overrideFlowContent]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, https://help.gong.io/docs/gong-engage-api-capabilities, scopes/gong-scopes.yml
---

# Manage Gong Engage flows and prospects

Requires Gong Engage. With a Bearer token these operations require the `api:flows:read` scope for reads; writes require the corresponding write scope on your registered integration.

## Steps

1. **Find the flow.** `listEngageFlows` — `GET /flows`. Returns company flows plus personal and shared flows belonging to the user named in `flowEmailOwner`. Flows in deleted or inaccessible folders are omitted. Visibility is one of Company, Personal or Shared.
2. **Browse folders if needed.** `listFlowFolders` — `GET /flows/folders`, same `flowEmailOwner` semantics.
3. **Check current state before writing.** `listAssignedFlowsForProspects` — `POST /flows/prospects`. Gives you the flows each prospect is already in, and the `flowInstanceId` for each assignment.
4. **Assign.** `assignProspectsToFlow` — `POST /flows/prospects/assign`. Send the `flowId`, a comma-separated list of CRM prospect ids (contacts or leads), and `flowInstanceOwnerEmail` — the Gong user who set up the flow instance and receives the to-dos. **Maximum 100 prospects per request.** A non-existent or deleted `flowId` returns `404`.
5. **Unassign.** `unassignProspectsFromFlow` — two addressing modes, and you must pick deliberately: by **CRM prospect id** (`/flows/prospects/unassign-flows-by-crm-id`) removes that person from flows; by **flow instance id** (`/flows/prospects/unassign-flows-by-instance-id`) removes one specific assignment. Use the instance id when a prospect is in several flows and you only mean to remove one.
6. **Optional content override.** `overrideFlowContent` — `PUT /flows/{flowId}/content-override` to tailor messaging for a campaign.

## Rules

- Always run step 3 before step 4 or 5. Assignment is not idempotent and Gong exposes no dry-run.
- Batch to 100 prospects and pace against 3 req/sec.
- `404` on assign means the flow is gone — do not retry, re-resolve the flow.
- Personal and shared flows are only visible via `flowEmailOwner`; omitting it silently narrows the result set to company flows.
