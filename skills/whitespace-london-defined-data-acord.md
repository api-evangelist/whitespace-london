---
name: Extract and write contract data with the Defined Data API (ACORD tagset)
description: >-
  Read structured London-market contract values out of a Whitespace risk against MRC headings or
  the ACORDGPM tagset, verify a data payload, and write tagged data back.
api: openapi/whitespace-london-platform-openapi.yml
generated: '2026-07-25'
method: generated
operations:
  - GET /api/v23.09/data/{riskID}
  - GET /api/v23.09/data/{riskID}/tagset/{tagset}
  - POST /api/v23.09/data/{riskID}/verify
  - POST /api/v23.09/data/{riskID}
  - GET /api/v23.09/data/questionnaire/{riskID}
  - GET /api/lookup/headings
  - GET /api/lookup/check/{heading}
  - GET /api/lookup/headingsInSection/{section}
  - GET /api/lookup/namevariations
  - GET /api/risks/{riskID}/getExtendedMRC
  - GET /api/risks/{riskID}/lineitem/{mrcHeading}
---

# Defined Data and the ACORD tagset

Defined Data is how Whitespace turns a placing contract into machine-readable values. Each value
is bound to a Market Reform Contract (MRC) heading, and can additionally be returned under an
alternative tagset — currently **ACORDGPM**, the tag family from ACORD's Global Reinsurance and
Large Commercial data standards. Whitespace authored the JSON Market Reform Contract (JMRC) and
donated it to ACORD, which is why the two vocabularies line up.

**Use v23.09.** `/api/v22.04/data/{riskID}` still works and always will — Whitespace does not
retire versioned endpoints — but its own documentation says new integrators should use v23.09.

## Read

1. **All tagged data, default tagset.** `GET /api/v23.09/data/{riskID}` returns the metadata
   block, the multi-section definition, the contract headings and the tagged `definedData`.
2. **ACORD tagset.** `GET /api/v23.09/data/{riskID}/tagset/ACORDGPM` returns the same values
   carrying ACORD tags — `ACORDGPM:InsuredName`, `ACORDGPM:BrokerContractReference`,
   `ACORDGPM:InceptionDate`, `ACORDGPM:ExpiryDate`, `ACORDGPM:InstructionType`,
   `ACORDGPM:InsuredAddressNumberAndStreet` and so on — alongside their `mrcHeading` values.
   This is the mapping to use when feeding a downstream ACORD-aligned system.
3. **Questionnaire answers.** `GET /api/v23.09/data/questionnaire/{riskID}` returns the answered
   questions on the risk.
4. **Single heading.** `GET /api/risks/{riskID}/lineitem/{mrcHeading}` returns the line item(s)
   under one contract heading.

## Validate the heading vocabulary

- `GET /api/lookup/headings` gives the enumerated MRC heading list.
- `GET /api/lookup/check/{heading}` tells you whether a heading you hold maps to an enumerated
  MRC heading.
- `GET /api/lookup/headingsInSection/{section}` and `GET /api/lookup/namevariations` resolve
  section membership and accepted name variants before you attempt a write.

## Write

5. **Verify first.** `POST /api/v23.09/data/{riskID}/verify` with the payload you intend to send.
   This is not optional in practice: an unacceptable payload on the write returns **500** with
   "The data was not acceptable. We recommend calling the … /verify POST endpoint first which
   will report the reason for rejection." The verify endpoint tells you *why*; the write does not.
6. **Write.** `POST /api/v23.09/data/{riskID}`. The payload needs the metadata from a previous
   GET plus a `multiSectionDefinition` array (which may be empty). Take the metadata from a
   fresh GET immediately before writing — stale document revisions are rejected.

## Rules that will bite you

- **Verify → write, always.** Skipping verify converts a data-quality problem into an opaque 500.
- **Round-trip the metadata.** The write payload is not free-form: it echoes the metadata from
  the immediately preceding GET.
- **No idempotency key.** A retried write is a second write attempt against a document whose
  `_rev` has moved on; re-GET before retrying.
- **`{riskID}` is the full instance ID** with `::` segments, not the bare root.
- Mandatory tag sets exist from platform 3.2 onwards: a contract can be required to be properly
  tagged before it may be shown as a firm order, so data writes can gate the placing flow.
