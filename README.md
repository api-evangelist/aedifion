# Aedifion

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

aedifion GmbH is a Cologne-based PropTech founded in 2017 that operates a vendor-neutral,
patented cloud platform for the optimized operation of non-residential buildings. It ingests
real-time operating data from all building trades via plug-and-play edge devices, applies
analytics and AI to detect faults, and autonomously controls HVAC plant based on weather
forecasts, occupancy and electricity prices.

- Website: https://www.aedifion.com/
- Developer documentation: https://docs.aedifion.io/en/developers/
- API console: https://api.aedifion.io/ui/

## API surface

| Surface | Where | Notes |
|---|---|---|
| HTTP API | `https://api.aedifion.io` | OpenAPI 3.0.1, 147 paths, **208 operations**, 239 schemas. Live at `/openapi.json` and `/openapi.yaml`. |
| MQTT API | `mqtt.aedifion.io:8883` / `:9001` | TLS-only broker, MQTT 3.1.1, InfluxDB line protocol. |
| Kafka | not published | SASL/SCRAM-SHA-512, topics `mtw.aed.{project_handle}.*`. Bootstrap servers are not published. |
| Alert notifications | via HTTP API | Outbound push to Microsoft Teams webhook URLs, Telegram, email, dashboard. |

Authentication is HTTP Basic (documented as legacy) or OpenID Connect bearer tokens from the
aedifion Keycloak realm at `https://auth.aedifion.io/realms/aedifion`.

## What this profile found

**Strengths.** The contract is unusually complete for this sector: every one of the 208
operations carries an `operationId`, a summary, a description and tags. The platform is ISO
27001 certified (since December 2023, certificate published) and hosted entirely in Germany.
The semantic component/pin data model aligns with Project Haystack, and BACnet unit
enumerations are mapped in the schema itself. A dated changelog is maintained and was updated
five days before this profile was written. aedifion also publishes a genuine, hand-written
`llms.txt`.

**Notable gaps.**

- The spec declares `servers: [{"url": ""}]` — an **empty string**. The Swagger UI works because
  the browser resolves it relative to the page, but any client generated from the downloaded
  document has no host to call. `info.title` is the generic "API Docs" and `info.version` is
  empty. See `overlays/aedifion-http-api-overlay.yaml`.
- **No client SDK exists in any package registry.** The only npm package, `aedifion-api`, was
  unpublished six minutes after it was published in 2021. The GitHub organisation's five repos
  were all last touched between 2018 and 2020, and the "open source Excel plugin" repository is
  empty.
- **No sandbox.** The interactive console calls production, where a setpoint write actuates real
  building equipment and 43 DELETE operations have no undo.
- No RFC 9457 problem details, no `Idempotency-Key`, no rate-limit headers, no ETags, no
  request-id header. Throttling is signalled inconsistently as 423 on one operation and 429 on
  another.
- No published vulnerability disclosure policy, `security.txt` or bug bounty.
- No MCP server. `mcp/aedifion-mcp.yml` holds a **candidate** tool set derived from real
  operationIds — it is a proposal, not a live agent surface.

**Where aedifion does better than most.** The highest-consequence operation on the API,
`post_datapoint_setpoint`, ships a real `dryrun` parameter, returns the prior value in
`SetpointAck.state_before`, and defines `value='null'` as a reset that hands the point back to
the local building automation system. That is a rehearsal path, an audit trail and a reversal
path on the one operation that moves physical plant.

## Artifacts

`openapi/` `asyncapi/` `authentication/` `scopes/` `conventions/` `errors/` `data-model/`
`conformance/` `lifecycle/` `changelog/` `rate-limits/` `plans/` `packages/` `components/`
`sandbox/` `security/` `well-known/` `llms/` `mcp/` `skills/` `overlays/`

Six packaged Agent Skills are in `skills/`; every `operationId` they reference was verified
against the live spec.
