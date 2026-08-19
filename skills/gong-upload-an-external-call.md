---
name: Upload an externally recorded call to Gong
description: Register a call recorded by another telephony or recording system with Gong, attach its media, and confirm Gong processed it — with the retry hazards spelled out, because Gong has no idempotency contract.
api: openapi/gong-calls-api-openapi.yml, openapi/gong-recordings-api-openapi.yml
operations: [addCall, addCallRecording, getCall]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conventions/gong-conventions.yml, errors/gong-problem-types.yml
---

# Upload an externally recorded call to Gong

Use this when calls are captured somewhere other than Gong and you want them transcribed, analyzed and attached to the deal timeline.

## Steps

1. **Register the call.** `addCall` — `POST /calls`. Send the metadata: `clientUniqueId`, `actualStart`, `parties[]` (each with `userId` for internal people or an email/phone for external ones), `direction`, `primaryUser`, `workspaceId`, and the audio channel your representative spoke on. The response returns `callId`.
2. **Attach the media.** `addCallRecording` — `PUT /calls/{id}/media` with the audio/video for the `callId` from step 1. Gong transcribes and analyzes only after media arrives.
3. **Confirm.** `getCall` — `GET /calls/{id}` to verify the call landed and, once processing completes, that parties and content are populated.

## Rules

- **`clientUniqueId` is a correlation id, not an idempotency key.** Gong documents no dedupe window and no `Idempotency-Key` header. If `addCall` times out or returns `429`, do **not** simply re-POST — first search for the call (`listCalls` over the window, matching `clientUniqueId`) and only create it if it is genuinely absent. A blind retry can create a duplicate call.
- Include the representative's channel in the audio. Gong's speaker separation depends on it; a single mixed mono channel degrades diarization and coaching stats.
- Uploaded calls do not identify speakers the way natively recorded calls do — expect thinner Team-page stats.
- `429` → honor `Retry-After`. `400` → the message is in `errors[]` with no field pointer; validate against the request schema.
- Budget: 3 req/sec, 10,000 req/day company-wide. A bulk backfill must be paced.
