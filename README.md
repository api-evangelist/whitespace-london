# Whitespace (whitespace-london)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
