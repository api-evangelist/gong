---
name: Push CRM data into Gong
description: Register a generic CRM integration, declare the object schema, upload objects, and poll the asynchronous upload to completion — the submit-then-poll flow Gong requires.
api: openapi/gong-crm-integration-api-openapi.yml, openapi/gong-crm-schema-api-openapi.yml, openapi/gong-crm-data-api-openapi.yml
operations: [registerCrmIntegration, getCrmIntegrations, uploadObjectSchema, getSchemaFields, uploadCrmObjects, getUploadStatus, getCrmObjects, deleteCrmIntegration]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conventions/gong-conventions.yml, data-model/gong-data-model.yml
---

# Push CRM data into Gong

Use this when the system of record is not Salesforce/HubSpot/Dynamics and you want accounts, contacts, leads and opportunities to appear alongside Gong conversations.

## Steps

1. **Register the integration.** `registerCrmIntegration` — `PUT /crm/integrations`. Returns an `integrationId`. Confirm with `getCrmIntegrations` — `GET /crm/integrations`.
2. **Declare the schema first.** `uploadObjectSchema` — `POST /crm/entity-schema` for each object type, referencing `integrationId`. Read it back with `getSchemaFields` — `GET /crm/entity-schema`. **Objects uploaded for a type with no declared schema will not map.**
3. **Upload the objects.** `uploadCrmObjects` — `POST /crm/entities`. This is **asynchronous**: the response returns a `clientRequestId`, not a result.
4. **Poll to completion.** `getUploadStatus` — `GET /crm/upload-status` with that request id. Do not assume success from the 200 on step 3.
5. **Verify.** `getCrmObjects` — `GET /crm/objects`.

## Rules

- Treat the whole flow as submit-then-poll. A 200 on `uploadCrmObjects` means *accepted*, not *applied*.
- Schema before data. Changing a schema after objects exist is a re-upload, not a migration.
- No idempotency contract: if `uploadCrmObjects` fails ambiguously, poll `getUploadStatus` for the `clientRequestId` you already hold **before** re-submitting.
- Pace the batches against 3 req/sec and 10,000 req/day.
- `deleteCrmIntegration` — `DELETE /crm/integrations` removes the integration; treat it as destructive and gate it behind human approval.
