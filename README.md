# Whitespace (whitespace-london)

Whitespace Software Limited is a London-based digital placing platform for the Lloyd's and London (re)insurance market, and one of only two electronic placing systems given fully recognised status by Lloyd's. Acquired by Verisk (via Sequel), Whitespace lets brokers and underwriters create, negotiate, quote, firm-order, line, sign and endorse fully digital subscription-market contracts in place of the traditional slip, across property and casualty, specialty, marine, aviation, energy and reinsurance business written in the United Kingdom and internationally.

Its API posture is unusually strong for the insurance sector but only half open: the reference documentation is genuinely public and unauthenticated at [apidocs.whitespace.co.uk](https://apidocs.whitespace.co.uk/), with a complete OpenAPI 3.0.0 description of 113 endpoints served through Swagger UI at [swagger.whitespace.co.uk](https://swagger.whitespace.co.uk/), yet there is no self-serve signup. Sandbox and production service tokens are issued by Whitespace Support to contracted broker, carrier and vendor organisations, are IP-allowlisted and expire within a year, so the integration surface itself is partner-gated.

Whitespace is ACORD-native rather than merely ACORD-aware: it authored the JSON Market Reform Contract (JMRC) standard and donated it to ACORD for incorporation into the Global Reinsurance and Large Commercial data standards, and its Defined Data API returns contract values tagged with an ACORDGPM tagset alongside MRC headings. Event integration is delivered as per-client Azure Service Bus queues rather than webhooks, and no AsyncAPI, GraphQL or public Postman workspace is published.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/whitespace-london/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/whitespace-london/refs/heads/main/apis.yml)

## Tags

- Insurance
- United Kingdom
- Reinsurance
- Property and Casualty
- Insurtech
- Broker
- Underwriting
- Placing Platform
- London Market
- Lloyd's of London
- ACORD
- Market Infrastructure

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## APIs

### Whitespace Platform API

A REST/JSON API over the full London-market subscription placing lifecycle, publicly described by an OpenAPI 3.0.0 document covering 113 endpoints across Risks, Data, Attachments, Documents, Activities, Comments, Labels, Lookup, Endorsements, Lines, Questionnaire, Summary, User and MI Report.

It genuinely exposes **quote** (`quoteOnRequest`, `saveQuoteDetails`, `getQuoteDetails`, `showQuoteToBroker`, `declineQuoteRequest`, `markAsIndicative`, `notTakeUpQuotation`) and **bind** (`createBindableQuote`, `draftToFirmOrder`, `markAsFirmOrder`, `writeLine`, `recordLine`, `requestWrittenLineSets`, `acceptWrittenLine`, `signSets`, `reSignSets`), plus **issue** of the contract document (`getExtendedMRC`, `export/pdf/{riskID}`, `documents/{docID}`) and a full post-bind endorsement lifecycle. It does **not** cover FNOL or claims — Whitespace is a placing platform, not a claims system.

The embedded Defined Data API (`/api/v22.04/data` and `/api/v23.09/data`) returns and verifies structured contract values tagged against MRC headings and an alternative `ACORDGPM` tagset.

- **Human URL:** [https://apidocs.whitespace.co.uk/](https://apidocs.whitespace.co.uk/)
- **Base URL:** `https://sandbox.whitespace.co.uk/`
- **Auth:** HTTP bearer JWT service token, issued by Whitespace Support, IP-allowlisted, one-year maximum lifetime. No self-serve key.

#### Properties

- [OpenAPI](openapi/whitespace-london-platform-openapi.yml) — harvested verbatim from `https://swagger.whitespace.co.uk/index.spec.js` on 2026-07-25 (HTTP 200)
- [API Reference](https://swagger.whitespace.co.uk/)
- [Documentation — API index](https://apidocs.whitespace.co.uk/)
- [Getting Started with the Whitespace API](https://apidocs.whitespace.co.uk/Getting_Started_with_the_Whitespace_API.pdf)
- [Obtaining and Using a Service Token for the API](https://apidocs.whitespace.co.uk/Obtaining_and_Using_a_Service_Token_for_the_API.pdf)
- [An Introduction to Defined Data](https://apidocs.whitespace.co.uk/An_Introduction_to_Defined_Data.pdf)
- [JMRC ACORD Specification v0.5 — Defined Data](https://apidocs.whitespace.co.uk/JMRC%20ACORD%20Specification%20V0.5%20-%20Defined%20Data.pdf)
- [Queues and the Whitespace Platform](https://apidocs.whitespace.co.uk/Queues_and_the_Whitespace_Platform_v1.2.2.pdf)

## Events

There are no webhooks. Every platform action creates an activity document that is placed on the client's own Azure Service Bus queue as a time-stamped message carrying an activity code, document reference and message id; the client system consumes it and calls back into the REST API. Queues are provisioned per organisation or team by Whitespace Support. No AsyncAPI document is published.

## Links

- [Website](https://www.whitespace.co.uk/)
- [Integration](https://www.whitespace.co.uk/integration/)
- [Support & Training](https://www.whitespace.co.uk/support-training/)
- [News & Events](https://www.whitespace.co.uk/news-events/)
- [Interchange Agreement](https://www.whitespace.co.uk/interchange-agreement)
- [LinkedIn](https://www.linkedin.com/company/whitespace-software-limited)
- [Verisk (parent)](https://www.verisk.com)
