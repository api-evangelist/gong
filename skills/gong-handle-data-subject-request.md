---
name: Handle a Gong data subject erasure request
description: Find everywhere Gong holds an email address or phone number, then erase it and everything associated with it — the GDPR/CCPA path, with the irreversibility called out.
api: openapi/gong-data-privacy-api-openapi.yml
operations: [retrieveEmailReferences, retrievePhoneReferences, deleteEmailAddressData, deletePhoneNumberData]
generated: '2026-08-13'
method: generated
source: openapi/*.yml, conformance/gong-conformance.yml, errors/gong-problem-types.yml
---

# Handle a Gong data subject erasure request

Gong exposes erasure as first-class API operations rather than a support ticket, which makes an automated DSAR pipeline possible. It is also destructive and has no undo.

## Steps

1. **Discover first — always.** `retrieveEmailReferences` — `GET /data-privacy/email-references`, or `retrievePhoneReferences` — `GET /data-privacy/phone-references`. Each returns `DataReference` records identifying every object holding that identifier. Capture this list; it is your only record of what erasure will touch.
2. **Report and get approval.** Present the reference list to the requesting human. Do not proceed autonomously.
3. **Erase.** `deleteEmailAddressData` — `POST /data-privacy/email-delete`, or `deletePhoneNumberData` — `POST /data-privacy/phone-delete`. This deletes the person and all associated elements, including calls they participated in.
4. **Verify.** Re-run step 1. An empty reference set is your completion evidence; persist it with the response `requestId` as the audit record.

## Rules

- **This is irreversible and there is no idempotency contract.** If the delete call fails ambiguously, re-run the *discovery* call to establish actual state before ever re-issuing the delete.
- Erasure removes conversations, not just the contact record. Confirm the requester understands the blast radius from step 1's output before step 3.
- Log `requestId` from every response — it is the only correlation handle Gong gives you and the only thing support can trace.
- `429` → honor `Retry-After`; never parallelize erasure calls.
