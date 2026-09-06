# EMBL-EBI Guidelines for Model Context Protocol (MCP) Servers

**Version 0.3 — pre-consultation draft (MCP 2026-07-28 baseline)**
**Date:** 6 September 2026
**Pinned MCP specification revision:** [`2026-07-28`](https://modelcontextprotocol.io/specification/2026-07-28) (current stable)
**Status:** Draft for discussion — *not yet ratified*
**Language:** British English

---

> **How to read this document**
>
> This draft proposes shared practices for EBI MCP servers. The working group has not approved them yet. Decisions marked **[OPEN]** are collected in [§11](#11-open-questions-for-the-working-group).
>
> **v0.3** includes the early reviewer feedback and uses MCP `2026-07-28`. [§2.4](#24-migrating-from-an-earlier-revision) explains the protocol changes since `2025-11-25`. This protocol update does not settle the working group's policy decisions.
>
> **v0.5** will be the first formally circulated working draft, at milestone M3 in the project charter. The document version and MCP protocol revision are separate.
>
> For a first read, start with the scope (§1), open decisions (§11) and checklist (§12). Sections 2–10 give the implementation details. **MUST** means required for conformance, **SHOULD** means recommended, and **MAY** means optional; §2 explains these terms.
>
> Comments are welcome as GitHub issues or pull-request suggestions against this file.

---

## Contents

1. [Purpose and scope](#1-purpose-and-scope)
2. [Normative baseline and conformance](#2-normative-baseline-and-conformance)
3. [Server identity, packaging and deployment](#3-server-identity-packaging-and-deployment)
4. [Tool naming and granularity](#4-tool-naming-and-granularity)
5. [Input and return-schema conventions](#5-input-and-return-schema-conventions)
6. [Error handling](#6-error-handling)
7. [Versioning and change management](#7-versioning-and-change-management)
8. [Discovery and registration](#8-discovery-and-registration)
9. [Authentication, authorisation and rate limiting](#9-authentication-authorisation-and-rate-limiting)
10. [Cross-resource links and shared vocabularies](#10-cross-resource-links-and-shared-vocabularies-deliverable-d4)
11. [Open questions for the working group](#11-open-questions-for-the-working-group)
12. [Conformance checklist](#12-conformance-checklist)
13. [References](#13-references)

---

## 1. Purpose and scope

EMBL-EBI teams are building MCP servers so AI applications can use their data and tools. Shared conventions will make these servers easier to build, maintain and use together.

These guidelines cover the choices teams need to make beyond the MCP specification, including tool names, data formats and security. Each resource team owns its server and roadmap.

**In scope:** tool names and design, input and output formats, errors, versioning, discovery, authentication, access permissions, rate limits, identifiers and shared vocabularies.

**Out of scope:** building or hosting MCP servers on behalf of other teams; extending the MCP protocol upstream; delivering a cross-resource agent product; and wider AI strategy questions owned by the AI Pilots portfolio or AI Guilds.

These are proposed guidelines, not a requirement to change resource roadmaps. We propose minimum security requirements in [§9](#9-authentication-authorisation-and-rate-limiting), subject to ITS/Security review before v1.0. The checklist describes what a server would need to meet to claim conformance with the guidelines.

---

## 2. Normative baseline and conformance

### 2.1 Pinned protocol revision

These guidelines use **MCP revision `2026-07-28`**, the current stable specification. Servers SHOULD implement this revision. Its protocol requirements apply whenever a server serves a request using this revision:

- Each request declares its protocol version. A server MUST reject an unsupported version with `UnsupportedProtocolVersionError` and list the versions it supports.
- Servers MUST implement `server/discover`. Clients MAY use it to find supported versions before sending other requests.

If these guidelines disagree with the specification, **follow the specification** and raise an issue here.

MCP `2026-07-28` treats each request independently. It does not use an initial handshake to agree a protocol version or establish a session. Supporting `2025-11-25` or earlier clients is optional. Servers claiming support for both approaches (**dual-era** support) MUST follow the specification's compatibility rules. Older clients cannot automatically switch to the new approach.

We will track upstream changes, but draft protocol changes do not alter these requirements. Adopting a later stable MCP revision will require a new version of these guidelines.

### 2.2 Requirement levels

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY** and **OPTIONAL** are to be interpreted as described in BCP 14 ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when, they appear in capitals.

### 2.3 Conformance levels

We propose three levels so teams can adopt incrementally:

| Level | Meaning |
|-------|---------|
| **Baseline** | Required for a server to claim conformance with these guidelines. Includes applicable MCP requirements and the proposed EBI minimum requirements. |
| **Recommended** | Strongly encouraged for interoperability and good agent behaviour. Expected for any server presented as a reference implementation. |
| **Aspirational** | Practices we expect to recommend or require in a future version. |

A requirement applies only where relevant: for example, HTTP headers do not apply to stdio. Optional features still have requirements when used. Use [§12](#12-conformance-checklist) to assess your server.

### 2.4 Migrating from an earlier revision

Moving from MCP `2025-11-25` to `2026-07-28` requires changes to how clients and servers communicate. The changes below apply to the **2026 protocol path**. A dual-era implementation also retains the behaviour required by the older revisions it supports.

- **Requests:** replace `initialize`, `notifications/initialized` and protocol sessions with metadata on every request. The required `_meta` keys are `io.modelcontextprotocol/protocolVersion` and `io.modelcontextprotocol/clientCapabilities`. Clients SHOULD include `io.modelcontextprotocol/clientInfo`; servers SHOULD include `io.modelcontextprotocol/serverInfo` in each result's `_meta`.
- **Version selection:** implement `server/discover`. Reject unsupported versions with `UnsupportedProtocolVersionError`. Follow the compatibility rules if supporting older clients too.
- **HTTP transport:** use one POST endpoint. Responses can still contain JSON or a Server-Sent Events (SSE) stream. The GET stream, `Mcp-Session-Id` and stream resumption using `Last-Event-ID` have been removed. Requests carry `MCP-Protocol-Version`, `Mcp-Method` and, for `tools/call`, `resources/read` and `prompts/get`, `Mcp-Name`. Servers MUST check that these headers match the request body.
- **Results and follow-up input:** every result carries `resultType`. Core values are `complete` and `input_required`. When a server needs more information, it returns `inputRequests`; the client retries the original request with `inputResponses`. This is the Multi Round-Trip Requests (MRTR) pattern, replacing separate server-initiated requests.
- **Caching:** completed discovery/list/read results MUST carry `ttlMs` (how long the result can be considered fresh) and `cacheScope` (whether it can be shared between callers). See §8.1 for the affected methods. List output SHOULD be in a consistent order.
- **Change notifications:** use `subscriptions/listen` instead of `resources/subscribe`, `resources/unsubscribe` or the GET stream. Progress and log messages stay on the response stream for the request they concern.
- **Tasks:** long-running work uses the optional `io.modelcontextprotocol/tasks` extension. Clients advertise support in each request's capabilities; servers advertise it through `server/discover`. Replace `tasks/result` with polling through `tasks/get`, use `tasks/update` to supply requested input, and remove `tasks/list`.
- **Schemas:** JSON Schema 2020-12 remains the default. Input and output schemas now allow the full keyword set. `structuredContent` can be any JSON value; when an `outputSchema` is declared, the value MUST conform to it.
- **Authorisation:** HTTP clients must check the authorisation server's identity and keep credentials tied to the server that issued them, as detailed in §9.2. Dynamic Client Registration is deprecated in favour of Client ID Metadata Documents.
- **Errors:** a missing resource in `resources/read` now uses JSON-RPC `-32602` instead of `-32002`. The range `-32020` to `-32099` is reserved for errors defined by MCP.
- **Removed methods and notifications:** remove `ping`, `logging/setLevel`, `notifications/roots/list_changed`, `notifications/elicitation/complete` and the URL-elicitation `elicitationId` field. If retaining MCP Logging, clients set the log level per request through `io.modelcontextprotocol/logLevel`; servers MUST NOT send log notifications without that field. Use `requestState` if an elicitation needs an identifier across retries.
- **Deprecated features:** Roots, Sampling and MCP Logging remain supported but new implementations SHOULD NOT adopt them. Use tool parameters or resource URIs for files, direct LLM provider APIs for model calls, and `stderr` or OpenTelemetry for logs. The older HTTP+SSE transport also remains deprecated.

Use the [upstream changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog) and [compatibility rules](https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning) for the full migration requirements.

---

## 3. Server identity, packaging and deployment

### 3.1 Naming the server

- Each server MUST have a stable, human-readable name that identifies the EBI resource (e.g. `Ensembl MCP`, `PDBe MCP`).
- For BioContextAI registration, use the stable `owner/repository` identifier. The official MCP Registry uses a verified reverse-DNS namespace instead (see [§8](#8-discovery-and-registration)).

### 3.2 Transport

- Remotely hosted servers SHOULD use **Streamable HTTP**: one MCP endpoint accepting independent POST requests, with JSON or SSE responses.
- Local or development servers MAY use **stdio**, which exchanges messages through the process's standard input and output.
- New servers MUST NOT adopt the deprecated HTTP+SSE transport. The 2026 protocol path MUST NOT rely on the removed GET stream, protocol sessions or SSE resumption. Dual-era servers retain legacy behaviour only for the older revisions they support (§2.1).
- Servers that need to preserve state between calls SHOULD return an opaque identifier (a **state handle**) for the client to pass back in later tool arguments. On authenticated servers, handles MUST be unguessable and bound to the verified caller. Knowing a handle is not proof of identity (see [§9](#9-authentication-authorisation-and-rate-limiting)).

### 3.3 Packaging and low-configuration setup

- Servers SHOULD be installable and runnable with one command and little configuration. For Python servers, publish to PyPI so users can run `uvx <package>`. A container image is RECOMMENDED for deployment in any language.
- Servers SHOULD ship an `mcp.json`-style client configuration snippet in their README.
- Non-secret configuration, such as endpoints and rate-limit settings, SHOULD be kept in a versioned configuration file where practical. It MUST NOT be hard-coded in source.
- Secrets, such as API keys and tokens, MUST come from the environment or a secret store. They MUST NOT be committed to the repository or placed in the versioned configuration file (see [§9.4](#94-secrets)).

### 3.4 Implementation

- Teams MAY use any language. For new Python servers, we suggest **FastMCP**, as used by the BioContextAI project template.
- In dynamically typed languages, tools SHOULD have complete type annotations so their input and output schemas can be generated and validated automatically.
- Check that the chosen library and version meet the protocol requirements and [§12 checklist](#12-conformance-checklist). Choosing a library does not by itself establish conformance.

**[OPEN]** Do we standardise on a single reference stack (FastMCP + container) for *new* EBI servers, or remain language-agnostic? See [§11](#11-open-questions-for-the-working-group).

---

## 4. Tool naming and granularity

Consistent tool names help people and AI applications identify tools from different resources.

### 4.1 Canonical format

- Tool names MUST follow the pinned specification's naming format (SEP-986). Teams MUST check their names against that specification.
- We propose the following EBI convention:
  - **`snake_case`** identifiers;
  - a resource prefix to distinguish tools when servers are combined, e.g. `ensembl_lookup_gene`, `pdbe_get_structure`, `pertcat_search_perturbations`;
  - a **`verb_object`** shape for the remainder (`get_`, `search_`, `list_`, `lookup_`, `map_`).
- Prefixes such as `intact_*` and `disprot_*` remain part of the tool name when tools are copied between clients or combined in one list.
- For tools that combine resources, we suggest no prefix or a neutral prefix, with the resources named in the description. This remains open for discussion in [§11.2](#11-open-questions-for-the-working-group).

**[OPEN]** Should names be distinguished by a resource prefix, by the server, or by an aggregator that combines servers? The decision also needs to cover tools that combine resources. See [§8](#8-discovery-and-registration) and [§11](#11-open-questions-for-the-working-group).

### 4.2 Granularity

Granularity means how much work each tool does. Early reviewers agreed that teams should avoid turning every REST endpoint into a separate tool. Design tools around what users need to do, and keep the set small enough for an agent to choose reliably.

Reviewers reported different results with two approaches:

- **Task-shaped tools:** specific operations such as finding variants in a region or mapping a gene to its orthologues. Each call is easy to describe and test. An early implementation reported good results with this approach. Avoid putting unrelated operations behind one free-text `query` tool with multiple modes.
- **A general query tool with maintained guidance:** a structured query interface, supported by instructions, examples and advice on handling failures. Another team's internal benchmarks favoured this approach over a fixed set of listing tools. It offers flexibility but needs continuing documentation and testing.

A server can offer both. The choice depends on its data model and the team's capacity to maintain the tools and guidance. These reports do not establish a default for all resources.

**[OPEN]** Which approach should we recommend as the default, or should we recommend both? See [§11.9](#11-open-questions-for-the-working-group).

### 4.3 Descriptions, titles and annotations

- Every tool MUST describe what it does, what it returns and its limits, including rate limits, maximum result sizes and any embargo rules.
- Tools SHOULD provide a human-friendly **title** and MAY provide **icons** (both supported in the pinned revision) for client display.
- Servers SHOULD set tool **annotations**, such as read-only or destructive hints, accurately. These help clients describe tools but do not enforce permissions.

---

## 5. Input and return-schema conventions

### 5.1 Inputs

- Every tool MUST declare a JSON Schema describing its inputs, with a description for each parameter. Implementations MUST support JSON Schema 2020-12 and SHOULD use it for new tools.
- Implementations MUST NOT automatically fetch schemas from network `$ref` addresses. They SHOULD limit schema depth, the number of subschemas and validation time to prevent excessive resource use.
- Inputs SHOULD accept identifiers as well as free text. For example, accept both a gene symbol and its canonical accession, and document which takes precedence.
- Servers using accessions SHOULD also accept CURIEs: identifiers with a namespace prefix, such as `uniprot:P04637`. Document the supported prefixes (see [§10.1](#101-identifiers)).
- Inputs MUST match their schema. Invalid tool arguments MUST produce a clear tool error (see [§6](#6-error-handling)).
- A server MAY use `x-mcp-header` to copy non-sensitive primitive inputs into HTTP headers for gateway routing or policy checks. Secrets, tokens, credentials and personal data MUST NOT appear in these headers.
- Servers taking accessions SHOULD check that each identifier resolves before making downstream calls. Return a clear not-found error if it does not. This **identifier-validation gate** catches plausible but invented identifiers. One early evaluation reported that this check eliminated fabricated-accession failures in its tests.

### 5.2 Structured outputs

- Tools SHOULD return machine-readable JSON in `structuredContent` and SHOULD declare an `outputSchema`. If a schema is declared, the server MUST provide structured results that conform to it. Tools returning structured content SHOULD also provide a text representation for older clients.
- Every result on the 2026 protocol path MUST carry `resultType`: `complete`, `input_required`, or a supported extension value. This describes the protocol result. The proposed EBI `status` field below describes the data returned by a tool.
- Structured results SHOULD include a field such as `status` to distinguish success, not-found and partial results. An empty list alone may be ambiguous.
- Tools on the same server SHOULD use a consistent outer structure, or **return envelope**. Put identifiers inside each data record. Put shared source information (**provenance**) once at the response level.

The following proposed envelope would go inside `structuredContent`. It is not a complete MCP response:

```jsonc
{
  "status": "ok",                          // outcome: "ok" | "not_found" | "partial"
  "data": [
    {
      "identifiers": {                     // per-result: one set per row
        "primary": "ensembl:ENSG00000139618",
        "xrefs": ["uniprot:P51587"]
      },
      "...": "result fields"
    }
  ],
  "provenance": {                          // per-response: one source per call
    "source": "Ensembl",
    "source_version": "release-114",
    "retrieved": "2026-06-18T10:00:00Z",
    "license": "https://creativecommons.org/publicdomain/zero/1.0/",
    "cite": "https://doi.org/10.1093/nar/..."
  },
  "pagination": { "total": 1234, "returned": 50, "truncated": true, "cursor": "..." }
}
```

Adopting this envelope is recommended and remains an open decision (§11.7). For servers that adopt it, the proposed field requirements are:

| Key | Level | Notes |
|-----|-------|-------|
| `status` | **mandatory** | Indicates success, not-found or a partial result. |
| `data` | **mandatory** on success | Array of results; each result carries its own `identifiers`. |
| `identifiers.primary` | **mandatory** per result | The primary identifier of that result. |
| `provenance` | **mandatory** on success | Omitted on error / not-found (see below). |
| `pagination` | mandatory for paginated tools | Include `truncated` and `returned` vs `total` (see [§5.4](#54-result-size-and-pagination)). |
| `identifiers.xrefs` | optional | Cross-references as CURIEs. |

In this envelope, omit data provenance on errors and not-found results because no data was retrieved. Keep shared provenance at the response level to avoid repeating it for every record. For large responses, allow callers to request fewer fields (§5.4).

### 5.3 Identifiers, provenance and FAIR

- Each returned data record MUST carry its primary identifier and SHOULD include cross-references as resolvable CURIEs, using the agreed prefix source (see [§10.1](#101-identifiers)).
- Successful responses SHOULD include provenance: the source resource, data version, retrieval time, data licence and citation. This is required when adopting the envelope in §5.2. Omit data provenance on errors and not-found results. Provenance helps users attribute and reproduce results, supporting FAIR data practices (Findable, Accessible, Interoperable and Reusable).
- Numeric values SHOULD state their units. Prefer a separate unit field, ideally using a unit CURIE, to a unit embedded in a label.

### 5.4 Result size and pagination

- Tools that can return large result sets MUST split them into pages and MUST document the default and maximum page size. Cursor-based pagination is RECOMMENDED: the response includes a cursor that the client passes back to request the next page.
- Tools SHOULD indicate when results have been cut short, using a `truncated` flag and `returned` and `total` counts. This lets callers distinguish a complete answer from a partial one.
- Tools SHOULD let callers request only the fields they need, to reduce response size.

---

## 6. Error handling

Errors should help the caller decide whether to retry, wait, ask the user or use another tool.

- Return protocol errors, such as malformed requests or unknown methods, through the JSON-RPC error channel. Return tool-execution errors, such as a failed upstream call, in a tool result with `isError: true`.
- Tool errors MUST include a stable machine-readable code and a clear message. Where possible, explain how to recover, for example: "No record for that accession; check the identifier namespace."
- Errors MUST NOT leak secrets, credentials, internal stack traces, or internal host/network detail.
- Rate-limit errors MUST be recognisable, with a retry-after indication where available. This lets clients wait before retrying (see [§9](#9-authentication-authorisation-and-rate-limiting)).
- If an upstream resource returns a rate-limit error, such as HTTP 429, the server SHOULD return a retryable tool error and preserve any `retry-after` hint. Distinguish this temporary limit from a permanent failure.
- For operations that may exceed client timeouts, servers MAY use the **Tasks extension** (`io.modelcontextprotocol/tasks`). Servers MUST advertise it through `server/discover` and MUST NOT return a task unless the client declares support in that request's capabilities. See §2.4 for migration details.
- Responses SHOULD be able to report partial success, identifying what succeeded and what failed.

---

## 7. Versioning and change management

- Servers MUST version their **software releases** with [Semantic Versioning](https://semver.org/) and maintain a changelog.
- Servers MUST implement `server/discover` and MUST handle the protocol version on each 2026 request correctly. Support for older revisions is optional. Claimed dual-era support MUST follow the specification's compatibility rules.
- Breaking changes to tool names, input schemas or output formats MUST follow a deprecation cycle: announce the change, support old and new formats for a stated period, then remove the old format.
- `server/discover` MUST list supported MCP revisions and SHOULD identify the server's software version. Clients can use this information to record which server they called.

---

## 8. Discovery and registration

### 8.1 Server instructions and metadata

- Servers MUST implement `server/discover` with accurate capabilities and supported versions. They SHOULD also include their name and software version.
- Completed results from `server/discover`, `tools/list`, `prompts/list`, `resources/list`, `resources/templates/list` and `resources/read` MUST include `ttlMs` and `cacheScope`. `ttlMs` MUST be non-negative; use `0` for an immediately stale result. `cacheScope` is `public` for results that can be shared or `private` for results restricted to the same authorisation context. Interim `input_required` results carry no caching hints. Follow the [MCP caching requirements](https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/caching).
- List results SHOULD use a consistent order so clients can cache them efficiently.
- Servers SHOULD provide short instructions through `server/discover`: what the resource offers, when to use it and any usage rules.
- Tool, resource and prompt metadata MUST give clients enough information to display and select them without separate documentation.
- A standard **Server Card**, a discovery document at a well-known web address, is still being developed and is not part of this MCP revision. Teams MAY experiment with a `/.well-known/` document, but MUST label it non-standard and MUST NOT describe it as a standard MCP feature.

### 8.2 Registration in BioContextAI and the official MCP Registry (Deliverable D2)

EBI MCP endpoints SHOULD be registered so clients can find them. This draft uses the **[BioContextAI Registry](https://biocontext.ai/registry)**, a community catalogue of biomedical MCP servers. Its entries use Schema.org and are available as JSON. To register, submit a pull request with a `meta.yaml` file, which the [online editor](https://biocontext.ai/registry/editor) can generate.

An illustrative EBI record using the registry [`schema.json`](https://github.com/biocontext-ai/registry/blob/main/schema.json) format follows. Replace example URLs and placeholders before submission:

```yaml
"@context": https://schema.org
"@type": SoftwareApplication
"@id": https://github.com/Ensembl/ensembl-mcp
identifier: Ensembl/ensembl-mcp
name: Ensembl MCP
description: >-
  Access to Ensembl gene, transcript, variation and comparative genomics data
  via the Model Context Protocol, for agentic queries over vertebrate and other
  genomes.
codeRepository: https://github.com/Ensembl/ensembl-mcp
url: https://mcp.ensembl.org/mcp/        # remote endpoint, if hosted
softwareHelp:
  "@type": CreativeWork
  url: https://github.com/Ensembl/ensembl-mcp#readme
maintainer:
  - "@type": Organization
    name: EMBL-EBI Ensembl
    url: https://www.ebi.ac.uk/about/teams/...
license: https://spdx.org/licenses/Apache-2.0.html
keywords: [genomics, ensembl, variants, comparative-genomics, ebi]
applicationCategory: ReferenceApplication
programmingLanguage: [Python]
featureList:
  - "Tool: ensembl_lookup_gene"
  - "Tool: ensembl_get_variants_in_region"
datePublished: 2026-06-18
```

BioContextAI requires a biomedical focus, free academic use, an OSI-approved open-source licence, MCP compliance, and public code and documentation. Duplicate entries need justification. These conditions inform the checklist in §12.

The **[official MCP Registry](https://modelcontextprotocol.io/registry/about)** covers all subject areas and is currently in preview. It uses `server.json` records and verified names based on reversed domain names. EBI teams MAY publish there as well as in BioContextAI. Publication in the official registry is not required by v0.3; the registry policy remains open in [§11.6](#11-open-questions-for-the-working-group).

### 8.3 EBI-internal inventory (Deliverable D1)

The project also maintains an internal inventory of current and planned EBI MCP work. It records endpoints, status (live, prototype or planned), available tools and blockers. The M1 survey provides the initial entries.

Some repositories may remain private during development. The inventory SHOULD allow a **"registration-ready, pending public"** status for servers with a validated `meta.yaml` awaiting publication. Build the inventory first, so the working group knows which servers could be combined through an aggregator.

**[OPEN]** Should EBI also provide an aggregator (a "meta-MCP") that presents several servers through one endpoint, or use BioContextAI and the internal inventory alone? See [§11](#11-open-questions-for-the-working-group).

---

## 9. Authentication, authorisation and rate limiting

> These proposed minimum security requirements need **ITS/Security review before v1.0**. AI applications may call services differently from users of existing web interfaces, so teams need to check their access controls and limits.

### 9.1 Default posture

- Public, read-only data MAY be exposed without authentication, but MUST be rate-limited (see §9.3).
- Streamable HTTP servers MUST validate the `Origin` header and reject a present, invalid origin with HTTP 403. Deployments SHOULD also check the `Host` header against their allowed hostnames. Local HTTP servers SHOULD listen only on the local machine (loopback).
- Servers MUST require authentication for tools that change data or other state, access controlled, embargoed or personal data, or use significant computing resources or money. Proposed triggers for a "costly" tool are more than a small number of upstream calls or a job lasting more than a few seconds. Tools combining resources (§10.4) need particular attention. The working group still needs to agree the thresholds.

### 9.2 How to authenticate (per the pinned revision)

The following OAuth requirements apply to **HTTP servers that require authentication**. Here, the MCP server checks access tokens; the authorisation server issues them.

- Protected HTTP servers MUST act as OAuth 2.1 Resource Servers, as defined by MCP.
- Clients MUST use **Resource Indicators** ([RFC 8707](https://www.rfc-editor.org/rfc/rfc8707)) to request tokens for the intended MCP server. Servers MUST check the token's audience and reject tokens intended for another server.
- Protected HTTP servers MUST publish OAuth Protected Resource Metadata so clients can discover how to authenticate. Their authorisation server MUST provide OAuth Authorization Server Metadata or OpenID Connect Discovery.
- EBI integrations SHOULD prefer Client ID Metadata Documents or pre-registration. Dynamic Client Registration is deprecated and SHOULD be retained only where needed for compatibility.
- The authorisation server SHOULD return its issuer identifier, `iss`. EBI clients MUST check a returned `iss` against the recorded issuer before exchanging an authorisation code for tokens. Clients MUST store credentials against the issuer that created them and MUST NOT reuse them with another issuer.
- MCP servers MUST NOT accept or pass through tokens that were not explicitly issued for them.

**stdio servers** SHOULD obtain credentials from the environment rather than implement the HTTP OAuth flow. A secret store can supply those environment values (§9.4). Follow the [MCP authorisation specification](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization) for transport-specific requirements.

The MCP host is responsible for obtaining explicit user consent before invoking tools. Servers MUST NOT assume consent or depend on an agent concealing an action from the user.

**[OPEN]** Which identity provider should controlled-access EBI servers use? Do we need a contact for ELIXIR's Authentication and Authorisation Infrastructure (AAI), as suggested in the charter for M2?

### 9.3 Rate limiting and abuse handling

- Every server MUST enforce rate limits and MUST signal them as a recognisable, retryable error (see [§6](#6-error-handling)), ideally with a retry-after hint.
- Servers SHOULD document their limits in tool descriptions and server instructions so well-behaved agents stay within them.
- Servers SHOULD have a way to respond to abuse, such as limiting requests per client or revoking access.

### 9.4 Secrets

- Servers MUST NOT log credentials, tokens or personal data. Secrets MUST come from the environment or a secret store and MUST NOT be committed to the repository or versioned configuration files.

---

## 10. Cross-resource links and shared vocabularies (Deliverable D4)

Shared identifiers and vocabularies let an agent follow links between resources in one workflow.

### 10.1 Identifiers

- Cross-references SHOULD be expressed as **Compact URIs (CURIEs)** with a registered prefix (e.g. `ensembl:ENSG00000139618`, `uniprot:P51587`, `efo:EFO_0000305`), so any consumer can resolve them.
- **[OPEN]** Which prefix registry should we use: **[identifiers.org](https://identifiers.org/)**, an EBI resource, or the **[Bioregistry](https://bioregistry.io/)**, which covers more namespaces? Agreeing a source helps servers interpret each prefix consistently. See [§11.10](#11-open-questions-for-the-working-group).
- Each server MUST document which identifier namespaces it accepts as input and which it returns. Servers SHOULD accept CURIE-form identifiers on input where applicable (see [§5.1](#51-inputs)).

### 10.2 Vocabularies

- Tools returning terms for diseases, tissues or assays SHOULD use a recognised ontology and include each term's identifier and label. The **EBI Ontology Lookup Service (OLS)** provides access to ontologies such as the **Experimental Factor Ontology (EFO)**. Identifiers let clients match terms across resources.

### 10.3 Worked example (to be hardened into the D4 reference)

An illustrative workflow follows BRCA2 across three resources using shared identifiers:

1. **Ensembl MCP** — `ensembl_lookup_gene(symbol="BRCA2")` → resolves to `ensembl:ENSG00000139618`, and returns an `xrefs` list including `uniprot:P51587`.
2. **Open Targets MCP** — `opentargets_get_associations(target="ensembl:ENSG00000139618")` → disease associations keyed on the *same* Ensembl identifier; diseases returned as `efo:` CURIEs.
3. **UniProt MCP** — `uniprot_get_entry(accession="uniprot:P51587")` → protein details, found by following the cross-reference.

The **Perturbation Catalogue** and the **Ensembl GraphQL proof-of-concept** are the two candidate reference implementations named in the charter; the production D4 example will be built on these.

### 10.4 Server-side cross-resource composites

In §10.3, the agent calls each resource in turn and combines the results. A server can also combine resources behind one tool call. Early implementers reported that agents sometimes missed steps in longer sequences.

- **Client-side traversal:** the agent chooses the sequence and can adapt it, but each step needs another call and may fail.
- **Server-side composite:** one tool handles a known sequence of calls and combines the results. This can improve reliability, but the team must maintain the integration with each resource. The tool MUST limit its upstream calls and apply the authentication rules in [§9.1](#91-default-posture).

Both approaches are valid. Name tools that combine resources using the convention in [§4.1](#41-canonical-format).

---

## 11. Open questions for the working group

The working group needs to decide the following questions, informed by the M1 survey and kick-off meeting.

1. **Federated aggregator.** Should EBI provide one endpoint combining its MCP servers, following the BioContextAI meta-MCP pattern, or use BioContextAI and the internal inventory alone? (§8)
2. **Tool namespacing.** Should tool names use resource prefixes (`ensembl_…`), server-level namespaces, or names assigned by the aggregator? Include tools that combine resources in this decision. (§4)
3. **Reference stack.** Should we recommend FastMCP and a container for new servers, or leave the language and libraries to each team? (§3)
4. **Mandatory vs recommended auth bar.** Which security requirements should be mandatory, subject to ITS/Security review? (§9)
5. **Identity provider.** Which authorisation server should controlled-access EBI servers use? Do we need an ELIXIR AAI contact? (§9)
6. **Registry of record.** Should BioContextAI be the only required biomedical registry, or should EBI servers also publish in the official MCP Registry once its preview stabilises? (§8)
7. **Return envelope.** Should we adopt the proposed response structure in §5.2, or allow more variation?
8. **Benchmark/metric.** The charter calls for a shared performance metric across EBI APIs (D3/M2/M3) — what do we measure, and how?
9. **Tool granularity.** Should we recommend tools for specific tasks, a general query tool with maintained guidance, or both? Reviewers reported different results with these approaches. (§4.2)
10. **CURIE prefix registry.** Should we use **identifiers.org** or the **Bioregistry** as the agreed source of identifier prefixes? (§10.1)

---

## 12. Conformance checklist

Use this checklist to assess a server against the proposed guidelines. **B** = Baseline (required), **R** = Recommended, **A** = Aspirational. Mark conditional items as not applicable where appropriate. Requirements for the 2026 protocol path do not change the behaviour needed for supported older revisions.

**Identity, packaging, deployment**

- [ ] (B) Stable, human-readable server name and appropriate registry identifier
- [ ] (R) Streamable HTTP for remote access; stdio permitted for local use
- [ ] (B) No deprecated HTTP+SSE transport in new servers; no removed transport behaviour on the 2026 protocol path
- [ ] (B) Required Streamable HTTP request metadata and routing headers validated
- [ ] (R) One-command install (`uvx …`) and/or container image
- [ ] (R) `mcp.json` client snippet in the README
- [ ] (B) No hard-coded configuration or secrets

**Tools and schemas**

- [ ] (B) Tool names conform to the spec's canonical format
- [ ] (R) EBI namespacing + `verb_object` shape *(pending §11.2)*
- [ ] (B) Every tool has input JSON Schema with parameter descriptions
- [ ] (B) JSON Schema 2020-12 supported; schemas are not automatically fetched from network `$ref` addresses
- [ ] (R) Schema depth, subschema count and validation time limited
- [ ] (B) Tool arguments validated against the input schema, with clear errors
- [ ] (B) Every tool has a clear description, including limits
- [ ] (R) Identifiers checked for existence before downstream calls
- [ ] (R) Structured (machine-readable) outputs, not prose-only
- [ ] (R) `outputSchema` declared
- [ ] (B) Where `outputSchema` is declared, structured results provided and conform to it
- [ ] (B) Every result on the 2026 protocol path carries an appropriate `resultType`
- [ ] (R) Consistent return envelope, with a field distinguishing success, not-found and partial results
- [ ] (B) Primary identifier on every data record; (R) cross-references as CURIEs
- [ ] (R) Provenance on success (source, version, time, licence, citation); required if adopting the §5.2 envelope; omitted on errors and not-found results
- [ ] (B) Large result sets paginated, with default and maximum page sizes documented
- [ ] (R) Truncation indicated with `truncated`, `returned` and `total`

**Errors and versioning**

- [ ] (B) Protocol vs tool errors on the correct channel
- [ ] (B) Structured, actionable, secret-free error payloads
- [ ] (B) Rate limiting signalled as a retryable error
- [ ] (R) Upstream 429 relayed as a retryable error with retry-after
- [ ] (B) Semantic Versioning + changelog
- [ ] (B) `server/discover` implemented with accurate supported versions and capabilities; (R) server name and software version included
- [ ] (B) Required version and client-capability metadata handled on each 2026 request, including unsupported-version errors
- [ ] (B) Compatibility rules followed if support for older, handshake-based clients is claimed
- [ ] (B) Completed discovery/list/read results listed in §8.1 include correct `ttlMs` and `cacheScope`
- [ ] (R) List results returned in a consistent order
- [ ] (B) If Tasks are used, the server advertises the extension and checks client support on each request
- [ ] (B) Deprecation cycle for breaking changes

**Security (proposed Baseline, pending ITS/Security review)**

- [ ] (B) Authentication required for changes to data or state, controlled data and costly operations
- [ ] (B) Streamable HTTP `Origin` validation; (R) deployed `Host` allowlist; (R) local HTTP binds to loopback
- [ ] (B) Protected HTTP servers act as OAuth 2.1 Resource Servers
- [ ] (B) Protected HTTP servers publish Protected Resource Metadata and support authorisation-server discovery
- [ ] (B) HTTP access tokens checked for the intended audience (RFC 8707); no token passthrough
- [ ] (B) EBI OAuth clients validate returned `iss` values and keep credentials tied to the issuer
- [ ] (R) HTTP integrations prefer Client ID Metadata Documents or pre-registration over deprecated Dynamic Client Registration
- [ ] (R) stdio credentials supplied through the environment, which may be populated from a secret store
- [ ] (B) On authenticated servers, state handles are unguessable and bound to the verified caller
- [ ] (B) Rate limiting enforced even when unauthenticated
- [ ] (B) No credentials, tokens or personal data in logs
- [ ] (B) Secrets supplied through the environment or a secret store, never committed to the repository

**Discovery, cross-resource, FAIR**

- [ ] (R) Server instructions provided
- [ ] (A) Experimental well-known discovery card clearly labelled non-standard until Server Cards stabilise
- [ ] (R) Registered in BioContextAI with valid `meta.yaml`
- [ ] (A) Published to the official MCP Registry while it remains in preview
- [ ] (B) Documented accepted/returned identifier namespaces
- [ ] (R) Ontology terms returned with IDs (OLS/EFO etc.)
- [ ] (B) OSI-approved open-source licence
- [ ] (B) Public repository and documentation
- [ ] (R) `CITATION.cff`; (R) usage examples; (R) test suite
- [ ] (B) British English; published openly on GitHub from the outset

---

## 13. References

- **MCP specification, revision 2026-07-28** — https://modelcontextprotocol.io/specification/2026-07-28
- **MCP 2026-07-28 changelog** — https://modelcontextprotocol.io/specification/2026-07-28/changelog
- **MCP deprecated-features registry** — https://modelcontextprotocol.io/specification/2026-07-28/deprecated
- **MCP versioning and compatibility** — https://modelcontextprotocol.io/specification/2026-07-28/basic/versioning
- **MCP Streamable HTTP transport** — https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http
- **MCP structured tool results** — https://modelcontextprotocol.io/specification/2026-07-28/server/tools
- **MCP caching requirements** — https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/caching
- **MCP authorisation** — https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization
- **MCP Tasks extension** — https://modelcontextprotocol.io/extensions/tasks/overview
- **Official MCP Registry** (preview) — https://modelcontextprotocol.io/registry/about
- **MCP security best practices** — https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices
- **Common Guidelines on MCPs — project (pilot) charter** — internal working-group document (available on request); the source for the deliverables (D1–D4) and milestones (M1–M5) referenced throughout.
- **BioContextAI** (Nature Biotechnology, 2025) — https://www.nature.com/articles/s41587-025-02900-9
- **BioContextAI Registry** — https://biocontext.ai/registry · repository: https://github.com/biocontext-ai/registry · schema: https://github.com/biocontext-ai/registry/blob/main/schema.json
- **BioContextAI MCP server cookiecutter** — https://github.com/biocontext-ai/mcp-server-cookiecutter
- **FastMCP** — https://gofastmcp.com/
- **RFC 2119 / RFC 8174** (requirement keywords) — https://www.rfc-editor.org/rfc/rfc2119 · https://www.rfc-editor.org/rfc/rfc8174
- **RFC 8707** (Resource Indicators) — https://www.rfc-editor.org/rfc/rfc8707
- **Bioregistry** — https://bioregistry.io/ · **identifiers.org** — https://identifiers.org/
- **EBI Ontology Lookup Service (OLS)** — https://www.ebi.ac.uk/ols4 · **EFO** — https://www.ebi.ac.uk/efo/
- **Semantic Versioning** — https://semver.org/

---

*Project: Common Guidelines on MCPs (EMBL-EBI). Co-sponsors: Fergal Martin, Henning Hermjakob. Coordinator: Alexey Sokolov. This document is published openly under the repository licence and welcomes contributions.*
