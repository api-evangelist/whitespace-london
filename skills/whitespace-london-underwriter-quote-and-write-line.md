---
name: Quote a risk and write a line as an underwriter
description: >-
  Respond to a broker's quote request on the Whitespace Platform — prepare and show a quote,
  set quote details, decline, and write or revise a line on a firm order.
api: openapi/whitespace-london-platform-openapi.yml
role: underwriter
generated: '2026-07-25'
method: generated
operations:
  - GET /api/user/myDetails
  - GET /api/summary
  - POST /api/summary
  - GET /api/risks/{riskID}
  - GET /api/risks/{riskID}/getExtendedMRC
  - POST /api/risks/{riskID}/changeDraftNowQuoteInPreparation
  - POST /api/risks/{riskID}/saveQuoteDetails
  - POST /api/risks/{riskID}/markAsIndicative
  - POST /api/risks/{riskID}/showQuoteToBroker
  - POST /api/risks/{riskID}/quoteOnRequest
  - POST /api/risks/{riskID}/declineQuoteRequest
  - GET /api/documents/getLineGuidanceForRoot/{rootID}
  - POST /api/risks/{riskID}/writeLine
  - POST /api/risks/{riskID}/declineFirmOrder
  - GET /api/lines/{rootID}/combinedSets
---

# Quote and write a line (underwriter)

You are acting for a carrier organisation on the Whitespace Platform. Everything here is
underwriter-only; a broker token will be rejected. Writing a line is a commitment to insure a
percentage of the contract — treat it as a binding act, never as an exploratory call.

## Find the work

- `GET /api/user/myDetails` confirms your organisation, role, teams and channels.
- `GET /api/summary` returns batches of risk summaries you can see (the sixty most recently
  updated by default; `?from=2024-01-01` narrows by date). `POST /api/summary` takes a filter
  payload instead. In production, drive this from the activity queue rather than polling:
  `Quote Requested` and `Requested a Line` are the two messages that mean work has arrived.
- `GET /api/risks/{riskID}` returns the contract in JMRC form;
  `GET /api/risks/{riskID}/getExtendedMRC` adds the questionnaire, written and signed line sets.

## Quote

1. **Open the request.** `POST /api/risks/{riskID}/changeDraftNowQuoteInPreparation` takes the
   broker's quotation request and starts an underwriter quote. This raises the
   `Quote in Preparation` activity.
2. **Add quote details (optional).** `POST /api/risks/{riskID}/saveQuoteDetails` fills the Quote
   Details pane — quoted line percentage, validity period and similar metadata. It is not part
   of the contract text.
3. **Flag it as indicative (optional).** `POST /api/risks/{riskID}/markAsIndicative` sets or
   unsets the indicative flag when you are expressing non-binding interest.
4. **Send it back.** `POST /api/risks/{riskID}/showQuoteToBroker` returns your prepared quote, or
   `POST /api/risks/{riskID}/quoteOnRequest` sends a quote straight back with no alterations to
   the request. Either produces `Quoted` or `Quoted with Quote Details` on the broker's queue.
5. **Or decline.** `POST /api/risks/{riskID}/declineQuoteRequest` declines to quote.

## Write the line

6. **Read the guidance first.** `GET /api/documents/getLineGuidanceForRoot/{rootID}` returns the
   percentages the broker is proposing for written lines, including any min/max constraints.
7. **Write.** `POST /api/risks/{riskID}/writeLine` commits your participation on the firm order
   or bindable quote. This is the binding step — confirm the risk is the intended instance
   (`…::FO`) and the percentage matches your authority before calling it.
8. **Or decline the firm order.** `POST /api/risks/{riskID}/declineFirmOrder`.
9. **Check the result.** `GET /api/lines/{rootID}/combinedSets` returns both written and signed
   line documents so you can confirm your line landed and later see the signed-down percentage.

## Rules that will bite you

- **`{rootID}` takes the bare root; `{riskID}` takes the full `::`-separated instance.** Getting
  this wrong is the leading cause of an unexplained 400.
- **Not idempotent.** There is no idempotency key. Writes require the current document `_rev`;
  a stale one is rejected. Re-GET, take the new `_rev`, then retry.
- **401 `Invalid WSAUTH`** means an expired/revoked token, a user not in Live state, a ReadOnly
  user attempting a POST, or a non-allowlisted IP address.
- **ReadOnly users cannot POST at all** — quoting and writing lines require a Live user or a
  dedicated service account.
- Your organisation may have Approved Brokers enabled, in which case requests from unapproved
  brokers surface as `Request Received from Unapproved Broker` rather than a normal request.
