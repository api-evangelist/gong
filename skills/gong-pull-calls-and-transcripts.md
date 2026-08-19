---
name: Pull Gong calls and transcripts
description: Retrieve a date range of Gong calls, expand the ones you care about with the extensive projection, and fetch their diarized transcripts — while staying inside Gong's 3 req/sec, 10,000 req/day budget.
api: openapi/gong-calls-api-openapi.yml, openapi/gong-transcripts-api-openapi.yml
operations: [listCalls, listCallsExtensive, getCall, getCallTranscripts]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conventions/gong-conventions.yml, errors/gong-problem-types.yml, rate-limits/gong-rate-limits.yml
---

# Pull Gong calls and transcripts

## Before you call anything

- Base URL is `https://api.gong.io/v2` for an Access Key credential. **If you hold an OAuth Bearer token, you must call the per-customer host returned as `api_base_url_for_customer` in the token response** (e.g. `https://company-17.api.gong.io`) — calling the generic host with a customer token is the most common 401.
- Auth is either `Authorization: Basic base64(accessKey:accessKeySecret)` or `Authorization: Bearer <token>`.
- Budget: 3 requests/second and 10,000 requests/day for the whole company. Plan the pull; do not fan out.

## Steps

1. **List the window.** `listCalls` — `GET /calls` with `fromDateTime` / `toDateTime`. This returns lightweight call metadata only.
2. **Page it.** The response carries `records` with `totalRecords`, `currentPageSize`, `currentPageNumber` and, when more remain, `cursor`. Repeat the identical request with `cursor` set to that value and every other input unchanged. Stop when `cursor` is absent.
3. **Expand only what you need.** `listCallsExtensive` — `POST /calls/extensive`. Send `filter` (by `callIds`, or by date range) and a `contentSelector.exposedFields` object naming only the blocks you want (`parties`, `content`, `trackers`, `topics`, `collaboration`, `media`). This is Gong's sparse-fieldset control and the main lever on payload size. Note `contentSelector.exposedFields.pointOfInterest` was removed on 2025-01-23 — set `highlights: true` and read `content.highlights` instead.
4. **Single call detail.** `getCall` — `GET /calls/{id}` when you only need one.
5. **Transcripts.** `getCallTranscripts` — `POST /calls/transcript` with `filter.callIds` (or a date range). Sentences carry a `speakerId`, which joins back to `parties[].speakerId` from step 3. **Transcript text alone does not tell you who spoke** — you must pull parties to resolve a speaker to a person.

## Rules

- **Never blind-retry a write.** This flow is read-only, so retries are safe here — but see the upload skill.
- On `429`, sleep for the `Retry-After` header value, then retry. There are no `X-RateLimit-*` headers, so track your own daily count against 10,000.
- Errors are `{ "requestId": "...", "errors": ["..."] }` — not RFC 9457. There are no error codes, so branch on HTTP status, not on message text, and log `requestId` for Gong support.
- `401` means credentials or wrong host. `403` means the API key has a Trusted IPs allowlist that does not include your egress address.
- Gong adds response fields without notice. Ignore unknown fields.
