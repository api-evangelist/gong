---
name: Audit Gong users, permissions and admin activity
description: Enumerate users and their managers, map permission profiles to the people who hold them, inspect who can reach which calls, and pull the audit log — the access-review flow.
api: openapi/gong-users-api-openapi.yml, openapi/gong-permission-profiles-api-openapi.yml, openapi/gong-audit-logs-api-openapi.yml, openapi/gong-workspaces-api-openapi.yml, openapi/gong-calls-api-openapi.yml
operations: [listWorkspaces, listUsers, listUsersByFilter, getUser, getUserSettingsHistory, listPermissionProfiles, getPermissionProfile, listUsersByPermissionProfile, retrieveUserCallAccess, giveUserCallAccess, removeUserCallAccess, retrieveAuditLogs]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, data-model/gong-data-model.yml, conventions/gong-conventions.yml
---

# Audit Gong users, permissions and admin activity

## Steps

1. **Enumerate workspaces.** `listWorkspaces` — `GET /workspaces`. Permission profiles and most other objects are workspace-scoped, so start here or you will audit one partition and think you covered the tenant.
2. **Enumerate users.** `listUsers` — `GET /users`, paging on `records.cursor`. Use `listUsersByFilter` — `POST /users/filter` for a targeted set. `User.managerId` is self-referential; build the reporting tree from it.
3. **Map profiles.** `listPermissionProfiles` — `GET /workspaces/{workspaceId}/permission-profiles`, then `getPermissionProfile` — `GET /permission-profiles/{profileId}` for the settings, then `listUsersByPermissionProfile` — `GET /permission-profiles/{profileId}/users` for the holders. The profile→user direction is the one that answers "who can do X".
4. **Check call-level grants.** `retrieveUserCallAccess` — `POST /calls/users-access` shows individual grants on specific calls, which sit *outside* the profile model and are the usual source of surprise access.
5. **Pull the log.** `retrieveAuditLogs` — `GET /logs` by type and time range. Entries carry `userId` and a polymorphic `resourceId`.
6. **Track drift.** `getUserSettingsHistory` — `GET /users/{id}/settings-history` for changes to an individual's data-capture settings.

## Rules

- `giveUserCallAccess` (`PUT /calls/users-access`) and `removeUserCallAccess` (`DELETE /calls/users-access`) are writes that change who can see recorded conversations. Gate them behind explicit human approval; there is no idempotency contract and no undo log beyond the audit trail.
- `resourceId` in an audit entry is polymorphic and unprefixed — you cannot infer the entity type from the id. Use the entry's type field.
- Repeat steps 3-4 per workspace.
- Pace against 3 req/sec, 10,000 req/day; a full-tenant access review of a large Gong instance can approach the daily cap.
