---
name: Build a queue-driven Whitespace integration
description: >-
  Consume the per-client Azure Service Bus activity queue, switch on the activity string, and
  call back into the Whitespace REST API to retrieve the full detail of what happened.
api: openapi/whitespace-london-platform-openapi.yml
events: asyncapi/whitespace-london-queues-asyncapi.yml
generated: '2026-07-25'
method: generated
operations:
  - GET /api/activities/{rootID}/full
  - GET /api/activities/{riskID}
  - GET /api/activities/filter/help
  - POST /api/activities/filter
  - GET /api/risks/root/{rootID}
  - GET /api/risks/{riskID}/getExtendedMRC
  - GET /api/documents/getLineGuidanceForRoot/{rootID}
  - GET /api/documents/{docID}
  - GET /api/attachments/{rootID}
  - POST /export/pdf/{riskID}
  - GET /api/shared/corporate
---

# Queue-driven integration

Whitespace has **no webhooks**. Its event mechanism is a per-client Microsoft Azure Service Bus
queue: every significant user action creates a JSON activity document that is placed on your
organisation's queue, time-stamped, and served oldest-first. Integrations are normally
*initiated* from the queue, and the REST API is used only to fetch the detail. Polling the REST
API instead is the wrong shape.

## Get a queue

Queues are provisioned by Whitespace Support (support@whitespace.co.uk), per organisation and
optionally per team. They supply the **queue name, key and connection string**. Request the
Sandbox queue as step two of the three-step integration process, then a Production queue.

## Consume

1. **Connect** with any standard Azure Service Bus client (.NET, Java, Python, JavaScript,
   TypeScript). Whitespace's own published sample uses `@azure/service-bus` with a settings file
   carrying `queueName` and `connectionString`.
2. **Receive, process, settle.** The oldest message is delivered and locked. Acknowledge it and
   the queue deletes it; crash or time out and it is unlocked and returned to the head of the
   queue. That is at-least-once delivery. For exactly-once, log processed message ids and screen
   incoming messages against the log — each message carries a unique id.
3. **Switch on `body.hardcodedActivity`.** That is the activity string identifying the precise
   action (`Quoted`, `Quoted with Quote Details`, `Requested a Line`, `Line Written`,
   `Signed Lines`, `Showed Endorsement`, `Quote Not Taken Up`, …). The full published
   enumeration is in `asyncapi/whitespace-london-queues-asyncapi.yml`. Route similar strings to
   one handler where it makes sense — `Quoted` and `Quoted with Quote Details` usually belong
   together.
4. **Check `type` first.** By default only `RWActivity` (and `RWCustomActivity`, if activated) is
   queued; `RWComment` chat documents are added only if your integration asked for them.

## Call back for detail

The message deliberately contains no sensitive contract data — it carries the reference, not the
contents. Use the IDs on the message:

- `parentDocID` and the first 38 characters of `_id` give you the contract instance and the root
  risk (`IC…` root, then `::FO` and similar stage segments).
- `GET /api/risks/root/{rootID}` — every document under that root risk.
- `GET /api/activities/{rootID}/full` — the complete time-stamped history of the risk;
  `GET /api/activities/{riskID}` for a single instance.
- `GET /api/documents/getLineGuidanceForRoot/{rootID}` — the percentages the broker proposed.
- `GET /api/risks/{riskID}/getExtendedMRC` — contract plus written and signed line sets.
- `GET /api/documents/{docID}` — any single platform document you are entitled to see.
- `GET /api/attachments/{rootID}` then `POST /export/pdf/{riskID}` — build a document pack for a
  contract library.
- `POST /api/activities/filter` (with `GET /api/activities/filter/help` for the vocabulary) —
  backfill or reconcile a specific activity type over a date range.

## Rules that will bite you

- **`{rootID}` accepts only the root form; `{riskID}` accepts the full `::` form.** The message
  gives you both — slice carefully.
- **Resolve users and organisations once.** `userID` values are `MU…` strings; map them with
  `GET /api/shared/corporate`, which lists every organisation, team, channel and member. The MU
  value for a user in a corporate identity is permanent, so cache it.
- **Channels tell you who can see what.** `channels[]` on the message uses the
  `companyid_TEAMID` form.
- **Messages never expire** and a queue holds more than a quarter of a million of them, so a
  consumer can safely process during downtime rather than at peak.
- Some activity strings in the published list are **reserved and not currently implemented**
  (`Viewed`, `Contract Updated`, `Risk Has Been Deleted`, …) — ignore rather than handle them.
