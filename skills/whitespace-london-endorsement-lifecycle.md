---
name: Run the endorsement lifecycle on a bound contract
description: >-
  Create, show, agree and complete a post-bind endorsement on a Whitespace contract, including
  follower roles, agreement templates and binding notify parties on a facility.
api: openapi/whitespace-london-platform-openapi.yml
generated: '2026-07-25'
method: generated
operations:
  - POST /api/risks/{riskID}/createEndorsement
  - POST /api/risks/{riskID}/editLineItem
  - POST /api/risks/{riskID}/setEndorsementFollowerRoles
  - POST /api/risks/{riskID}/setEndorsementFollowerTemplate
  - POST /api/risks/{EndorsementID}/showEndorsement
  - POST /api/risks/{EndorsementID}/acceptEndorsement
  - POST /api/risks/{riskID}/completeEndorsement
  - POST /api/risks/{riskID}/deleteEndorsement
  - POST /api/bindNotifyParties
  - GET /api/endorsements/{rootID}
  - GET /api/risks/{riskID}/endorsementFollowers
  - GET /api/activities/{rootID}/full
---

# Endorsement lifecycle

An endorsement changes a contract after it has been agreed. On Whitespace it is a separate
document instance under the same root risk, and it runs its own miniature placing cycle: create,
edit, show, accept, complete. Brokers drive it; underwriters accept or decline.

## Steps

1. **Create.** `POST /api/risks/{riskID}/createEndorsement` creates a new endorsement against the
   firm-order or signed contract (broker only). It appears under the same `IC…` root with its own
   instance segment.
2. **Draft the wording.** `POST /api/risks/{riskID}/editLineItem` (or `editMultipleLineItems`)
   while it is still unshown. Unshown endorsements are the only editable ones; delete a mistake
   with `POST /api/risks/{riskID}/deleteEndorsement`.
3. **Set who agrees.** `POST /api/risks/{riskID}/setEndorsementFollowerRoles` sets the carriers'
   roles on the endorsement, and `POST /api/risks/{riskID}/setEndorsementFollowerTemplate` sets
   the agreement template. Read the current state with
   `GET /api/risks/{riskID}/endorsementFollowers`.
4. **Show it.** `POST /api/risks/{EndorsementID}/showEndorsement` shows the endorsement to the
   underwriters (broker only). Note the parameter is the endorsement's own document ID.
5. **Accept.** The underwriter calls `POST /api/risks/{EndorsementID}/acceptEndorsement`. A
   decline or a withdrawal shows up on the queue as `Declined Endorsement` or
   `Withdrew Endorsement`.
6. **Bind non-agreement parties.** Where the contract is declared to a facility or line slip,
   `POST /api/bindNotifyParties` binds the notify and non-notify parties following the lead
   underwriter's acceptance.
7. **Complete.** `POST /api/risks/{riskID}/completeEndorsement` finalises the endorsement and
   binds any remaining notify parties. `GET /api/endorsements/{rootID}` lists every endorsement
   on the risk; `GET /api/activities/{rootID}/full` gives the audit trail.

## Rules that will bite you

- **Two different ID parameters.** `{riskID}` is the contract instance; `{EndorsementID}` is the
  endorsement document. Both are full `::`-separated IDs, not the bare root — only
  `GET /api/endorsements/{rootID}` takes the root.
- **An endorsement can only be withdrawn before acceptance.** After acceptance the route is a
  further endorsement or a contract correction
  (`POST /api/risks/{riskID}/createContractCorrection`).
- **No idempotency key.** Every write requires the current `_rev`; re-GET before any retry.
  Calling `completeEndorsement` twice is not a no-op.
- **Mid-term participant changes** complete as their own activity
  (`Mid-Term Participant Change Completed`) — watch for it on the queue rather than assuming
  `Completed Endorsement`.
- Errors are largely opaque 400s; 401 is `{"error": true, "reason": "Invalid WSAUTH"}`. See
  `errors/whitespace-london-problem-types.yml`.
