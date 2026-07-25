---
name: Place and sign a London-market risk as a broker
description: >-
  Take a subscription-market contract from a new draft through quote request, firm order,
  written lines and signing on the Whitespace Platform, acting for a broker organisation.
api: openapi/whitespace-london-platform-openapi.yml
role: broker
generated: '2026-07-25'
method: generated
operations:
  - GET /api/version
  - GET /api/user/myDetails
  - POST /api/risks/newDraft
  - POST /api/risks/save
  - POST /api/risks/{riskID}/editLineItem
  - POST /api/risks/{riskID}/showToLeadUnderwriter
  - GET /api/risks/{riskID}/getQuoteDetails
  - POST /api/risks/{riskID}/markAsFirmOrder
  - POST /api/risks/{riskID}/draftToFirmOrder
  - POST /api/risks/{riskID}/requestWrittenLineSets
  - GET /api/documents/getLineGuidanceForRoot/{rootID}
  - GET /api/lines/{rootID}/combinedSets
  - POST /api/risks/{riskID}/acceptWrittenLine/{writtenLineSetID}
  - POST /api/risks/{riskID}/signSets
  - GET /api/risks/{riskID}/getExtendedMRC
---

# Place and sign a risk (broker)

Whitespace is the digital slip for the Lloyd's and London subscription market. Every step below
is a real market act with legal consequences — a firm order is an offer to contract, and
`signSets` finalises the contract. Never call a write operation speculatively.

## Before you start

- Base URL: `https://sandbox.whitespace.co.uk` for testing, `https://www.whitespaceplatform.com`
  for the live platform. Build and test everything on Sandbox first.
- Auth: `Authorization: Bearer <service token>`. Tokens are issued by Whitespace Support, are
  IP-allowlisted and expire within a year. See `authentication/whitespace-london-authentication.yml`.
- Confirm connectivity with `GET /api/version` (no auth required), then confirm identity, role
  and channels with `GET /api/user/myDetails`. If the account is SUMO (multiple corporate
  identities) you must also send the `UserID: MU…` header on every call.
- These operations are broker-only. An underwriter token will be rejected.

## Steps

1. **Create the contract.** `POST /api/risks/newDraft` for the simplified input format, or
   `POST /api/risks/save` to create a risk from a full JMRC document. Keep the returned root ID
   (`IC…`, 38 characters) — every later call hangs off it.
2. **Edit the contract text.** `POST /api/risks/{riskID}/editLineItem` (or
   `POST /api/risks/{riskID}/editMultipleLineItems`) to write contract headings and their text.
   Only unshown drafts are editable. Every edit needs the current document revision (`_rev`);
   a stale or blank `_rev` is rejected.
3. **Show it for quote.** `POST /api/risks/{riskID}/showToLeadUnderwriter` sends the draft to the
   underwriter(s) you name. This is visible to the market — it cannot be undone silently.
4. **Read the quote back.** When the underwriter responds, `GET /api/risks/{riskID}/getQuoteDetails`
   returns anything they entered in the Quote Details pane, and
   `GET /api/risks/{riskID}/getExtendedMRC` returns the contract with its line sets.
5. **Go to firm order.** `POST /api/risks/{riskID}/markAsFirmOrder` converts the underwriter's
   quote into the contract of insurance, or `POST /api/risks/{riskID}/draftToFirmOrder` promotes
   your own draft. The firm-order instance appears under the root ID as `…::FO`.
6. **Ask the market to write lines.** Publish your line guidance and then
   `POST /api/risks/{riskID}/requestWrittenLineSets` to send the firm order to carriers. Check
   guidance with `GET /api/documents/getLineGuidanceForRoot/{rootID}`.
7. **Track and accept lines.** `GET /api/lines/{rootID}/combinedSets` returns both written and
   signed line documents. Where an underwriter attached subjectivities, respond with
   `POST /api/risks/{riskID}/acceptWrittenLine/{writtenLineSetID}` or
   `POST /api/risks/{riskID}/rejectWrittenLine/{writtenLineSetID}`.
8. **Sign.** `POST /api/risks/{riskID}/signSets` signs the written lines and promotes the risk to
   signed status. This is the point of no return; a correction afterwards requires
   `POST /api/risks/{riskID}/rollbackSigned` or `POST /api/risks/{riskID}/reSignSets`.

## Rules that will bite you

- **Root vs instance IDs.** `{rootID}` parameters take only the 38-character root
  (`IC213DA609-…`). `{riskID}` and `{docID}` take the full instance form with `::` segments
  (`IC213DA609-…::FO`). Using the wrong one is the most common cause of an opaque 400.
- **No idempotency key.** Re-POSTing a step is NOT safe. Concurrency is controlled by the
  document revision `_rev`, which must be the most recent one. On failure, re-GET the document,
  take the current `_rev`, and re-submit — do not blind-retry.
- **Errors are deliberately opaque.** Most failures are a bare 400 with no reason, by design.
  401 returns `{"error": true, "reason": "Invalid WSAUTH"}` — check token expiry, user state
  (must be Live, not ReadOnly for POSTs) and IP allowlisting. See
  `errors/whitespace-london-problem-types.yml`.
- **Channels govern visibility.** You only see documents carrying one of your
  `companyid_TEAMID` channels. Resolve counterparty channels with `GET /api/shared/corporate`.
- Do not confuse `teamID` (upper case, `MARINE`) with a channel (`priceforbes_MARINE`).

## Confirming what happened

Do not poll. The platform pushes an activity message to your Azure Service Bus queue for every
step above (`Created New Draft`, `Quote Requested`, `Marked as Firm Order`, `Requested a Line`,
`Line Written`, `Signed Lines`). Consume the queue and call back into the API — see
`skills/whitespace-london-queue-driven-integration.md`.
