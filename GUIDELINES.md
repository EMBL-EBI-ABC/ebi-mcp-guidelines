# EMBL-EBI Guidelines for Model Context Protocol (MCP) Servers

**Version 0.2 — pre-consultation draft (first-round feedback incorporated)**
**Date:** 14 July 2026
**Pinned MCP specification revision:** [`2025-11-25`](https://modelcontextprotocol.io/specification/2025-11-25) (current stable)
**Status:** Draft for discussion — *not yet ratified*
**Language:** British English

---

> **How to read this document**
>
> This is a **strawman**, circulated to seed the working-group survey and the first meeting. It is deliberately opinionated so that reviewers have something concrete to push back on. Nothing here is settled. Sections marked **[OPEN]** are decisions we are explicitly deferring to consultation, and are collected in [§11 Open questions](#11-open-questions-for-the-working-group).
>
> **v0.2** folds in the first round of reviewer feedback (June–July 2026). Where that feedback surfaced a genuine disagreement — tool granularity, and which CURIE prefix registry is authoritative — the point is captured as a *new open question* rather than resolved unilaterally, keeping the strawman honest.
>
> The first *formally circulated* working draft is still **v0.5** (per the project charter, milestone M3). This pre-consultation line (v0.1 → v0.2) exists only to get us there.
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
10. [Cross-resource links and shared vocabularies](#10-cross-resource-links-and-shared-vocabularies)
11. [Open questions for the working group](#11-open-questions-for-the-working-group)
12. [Conformance checklist](#12-conformance-checklist)
13. [References](#13-references)

---

## 1. Purpose and scope

EMBL-EBI service teams are increasingly asked to expose their data and tools to LLM-based agents via the Model Context Protocol (MCP). Several teams have already begun building MCP servers independently, and demand from consortia and industry partners is growing. Without shared conventions, each resource will invent its own approach to naming, schemas, authentication, error handling and discovery — fragmenting the agentic-access experience and making cross-resource workflows brittle.

These guidelines give EBI resource teams a common reference to apply when building their own MCP server, so that servers across EBI are consistent and interoperable. The aims are to lower the per-resource cost of MCP adoption, establish a security and governance baseline appropriate to public bioinformatics infrastructure, and position EBI as a reference site for agentic bioinformatics.

**In scope:** conventions for tool naming and granularity, return schemas, error handling, versioning, discovery, the authentication/authorisation posture, rate limiting, and cross-resource identifier and vocabulary conventions.

**Out of scope:** building or hosting MCP servers on behalf of other teams; extending the MCP protocol upstream; delivering a cross-resource agent product; and wider AI strategy questions owned by the AI Pilots portfolio or AI Guilds.

These guidelines are **recommendations for EBI resource teams**. They do not mandate changes to any resource roadmap. The one area where we propose a hard floor rather than a recommendation is the security baseline in [§9](#9-authentication-authorisation-and-rate-limiting), and even that is subject to ITS/Security review before v1.0.

---

## 2. Normative baseline and conformance

### 2.1 Pinned protocol revision

These guidelines are written against **MCP revision `2025-11-25`**, the current stable specification. Servers SHOULD implement this revision and MUST correctly perform protocol version negotiation so that older clients continue to function. Where these guidelines and the specification disagree, **the specification wins**; please raise an issue so we can correct the guidelines.

Upstream protocol changes during the project window will be **tracked, not silently absorbed**. A release candidate (`2026-07-28`) is in draft at the time of writing; we are not building against it. When the protocol moves, we will issue a revised version of these guidelines (v1.1, v2.0) rather than editing in place.

### 2.2 Requirement levels

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY** and **OPTIONAL** are to be interpreted as described in BCP 14 ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when, they appear in capitals.

### 2.3 Conformance levels

We propose three levels so teams can adopt incrementally:

| Level | Meaning |
|-------|---------|
| **Baseline** | Every EBI MCP server MUST meet these. Primarily the security, identity and documentation requirements. |
| **Recommended** | Strongly encouraged for interoperability and good agent behaviour. Expected for any server presented as a reference implementation. |
| **Aspirational** | Practices we expect to promote to *Recommended* or *Baseline* in a future version. |

A self-assessment checklist is provided in [§12](#12-conformance-checklist).

### 2.4 Migrating from an earlier revision

Teams that began against an earlier revision (e.g. `2025-06-18`) pay a real, additive cost to reach the pinned `2025-11-25`. The items **new in `2025-11-25`** — so a migrating server needs to *add* them rather than already having them — include:

- the **SEP-986 canonical tool-name format** (verify existing names against it — see [§4.1](#41-canonical-format));
- the OAuth **Client ID Metadata Documents** and **OpenID Connect Discovery** additions to the authorisation flow (see [§9.2](#92-how-to-authenticate-per-the-pinned-revision));
- the experimental **Tasks** mechanism for long-running operations (see [§6](#6-error-handling)).

This is a migration aid, not a criticism of the pin: a team already on `2025-11-25` has nothing to do here.

---

## 3. Server identity, packaging and deployment

### 3.1 Naming the server

- Each server MUST have a stable, human-readable name that identifies the EBI resource (e.g. `Ensembl MCP`, `PDBe MCP`).
- The canonical identifier for registration is `owner/repository` (see [§8](#8-discovery-and-registration)).

### 3.2 Transport

- Remotely hosted servers SHOULD use the **Streamable HTTP** transport defined in the pinned revision.
- Local or development servers MAY use the **stdio** transport.
- Servers MUST NOT rely on transports removed from the pinned revision.

### 3.3 Packaging and low-configuration setup

- Servers SHOULD be installable and runnable with a single command and minimal configuration. For Python servers this means publishing to PyPI so the server runs via `uvx <package>`; a container image is RECOMMENDED for language-agnostic deployment.
- Servers SHOULD ship an `mcp.json`-style client configuration snippet in their README.
- **Non-secret configuration** (endpoints, rate-limit tiers, feature switches) SHOULD be supplied via a **checked-in configuration file** rather than environment variables where practical — a file is clearer, reviewable and versionable. It MUST NOT be hard-coded in source.
- **Secrets** (API keys, tokens, credentials) are the deliberate exception: they MUST come from the environment or a secret store and MUST NOT be committed to the repository or placed in the versioned config file (see [§9.4](#94-secrets)). Keeping the two apart is the point — the config file is meant to be readable in the open; secrets are not.

### 3.4 Implementation

- Teams MAY use any language. For new Python servers we suggest **FastMCP**, consistent with the BioContextAI cookiecutter template, to reduce boilerplate and keep behaviour consistent across EBI.
- In dynamically typed languages, tools SHOULD be **fully typed** so that input/output schemas are generated and validated automatically.
- The real contract is the **conformance checklist** ([§12](#12-conformance-checklist)), not any specific library: a server in any language that meets the checklist is conformant. We recommend FastMCP for Python teams as a cheap way to get there, without mandating a stack that would exclude non-Python teams for no conformance gain.

**[OPEN]** Do we standardise on a single reference stack (FastMCP + container) for *new* EBI servers, or remain language-agnostic? See [§11](#11-open-questions-for-the-working-group).

---

## 4. Tool naming and granularity

Consistent tool names are the single highest-leverage interoperability win: they let agents (and humans) discover, display and reason about tools across resources without bespoke glue.

### 4.1 Canonical format

- Tool names MUST conform to the canonical tool-name format defined by the pinned revision (SEP-986). Teams MUST verify their names against the specification rather than against this paragraph.
- As an EBI house convention on top of the spec, we propose:
  - **`snake_case`** identifiers;
  - a **resource namespace prefix** so names are globally unambiguous when many EBI servers are aggregated — e.g. `ensembl_lookup_gene`, `pdbe_get_structure`, `pertcat_search_perturbations`;
  - a **`verb_object`** shape for the remainder (`get_`, `search_`, `list_`, `lookup_`, `map_`).
- Prefixed names (`intact_*`, `disprot_*`) also survive being **copied between clients and flattened into a single tool list**, where namespacing injected by an aggregator does not — a further reason to prefer them.
- **Cross-resource tools that join two resources have no natural single prefix.** Pending [§11.2](#11-open-questions-for-the-working-group), we suggest leaving such join tools **unprefixed** (or under a neutral prefix) and naming the resources they span in the description, so teams converge on one convention rather than each inventing their own.

**[OPEN]** The namespacing scheme (prefix vs server-level namespace vs relying on the aggregator), *including how cross-resource join tools are named*, is a working-group decision. It interacts with how a federated aggregator presents tools — see [§8](#8-discovery-and-registration) and [§11](#11-open-questions-for-the-working-group).

### 4.2 Granularity

One point of firm agreement, and one genuine open debate surfaced by reviewers.

**Agreed:** do **not** mirror REST routes 1:1. A tool surface that simply re-exposes every upstream endpoint recreates the REST API an agent could already call, and adds little. Tools should be shaped around what an agent wants to accomplish, and a **small number of well-described tools** is easier for an agent to choose between (and cheaper in context) than a large, sprawling surface.

**Under debate — two designs, both defensible:**

- **(a) Task-shaped tools.** A handful of clear, composable operations, each wrapping roughly one upstream operation ("find variants in a region", "map a gene to its orthologues"). Easy for an agent to select; each call's shape is explicit and testable. One reviewer building an IntAct + DisProt server reports these "held up well". The failure mode to avoid at this end is a single overloaded free-text `query` tool with a mode-switch, which is hard to use reliably.
- **(b) A generic query tool plus curated usage context.** A single expressive query interface (e.g. over a structured API or query language), shipped with carefully curated instructions, examples and failure guidance drawn from the team's own benchmarking. The argument — made from Open Targets' internal benchmarking, where a general query tool outperformed an explicit set of listing tools by a wide margin — is that a curated set of fixed tools effectively recreates fixed endpoints, whereas a well-documented generic query tool is more expressive and better aligned with how an LLM composes a request. The cost is that it takes real, ongoing curation and benchmarking to stay reliable.

The two are not strictly opposed: a server may offer a small set of task-shaped tools *and* a generic query escape hatch. Which to lead with is genuinely unsettled and depends on the resource's data model and how much curation the team can sustain.

**[OPEN]** Task-shaped tools vs a generic query tool + curated context, as the recommended default — see [§11.9](#11-open-questions-for-the-working-group).

### 4.3 Descriptions, titles and annotations

- Every tool MUST have a clear natural-language description stating what it does, what it returns, and any important limits (rate limits, maximum result sizes, embargo rules).
- Tools SHOULD provide a human-friendly **title** and MAY provide an **icon** (both supported in the pinned revision) for client display.
- Where the pinned revision supports tool **annotations** (e.g. read-only / destructive hints), servers SHOULD set them honestly. Note that clients treat annotations from untrusted servers with caution — they aid display, they are not a security control.

---

## 5. Input and return-schema conventions

### 5.1 Inputs

- Every tool MUST declare a JSON Schema for its inputs, with descriptions on each parameter.
- Inputs SHOULD use **identifiers**, not only free text. Where an agent might pass a gene symbol, accept the symbol *and* the canonical accession, and document which is authoritative.
- Where a resource is keyed on accessions, servers SHOULD also **accept identifiers in CURIE form** on input (an agent will send `uniprot:P04637`) and document which namespaces they parse (see [§10.1](#101-identifiers)).
- Inputs MUST validate against their schema; invalid input MUST produce a clear tool error (see [§6](#6-error-handling)), not a 200-with-garbage.
- **Accepting a well-formed identifier is not the same as confirming it resolves.** LLMs fabricate plausible-looking accessions, and an accept-only tool can pass a fabrication straight to the upstream API, which then returns an empty or wrong result the agent treats as real. Servers taking accessions SHOULD add a cheap **identifier-validation gate** (RECOMMENDED): a preflight that *resolves and confirms* the identifier before any downstream call, returning a clear not-found error if it does not resolve. In one reviewer's evaluation, making this gate mandatory removed fabricated-accession failures entirely. It generalises to any accession-keyed resource and is a good candidate for a shared recipe.

### 5.2 Structured outputs

- Tools SHOULD return **structured content** (machine-readable JSON), in addition to any human-readable text, using the structured-output mechanism of the pinned revision. Returning only a prose blob forces the agent to re-parse and is fragile.
- Structured results SHOULD carry a **discriminator** so *success*, *not-found* and any result variants are distinguishable without guessing (e.g. a `status` field), rather than overloading an empty list to mean several different things.
- Return payloads SHOULD use a **consistent envelope** across a server, so an agent learns the shape once. Two cardinalities are deliberately kept apart: **per-result identifiers** (one set per row) live *inside* each result, while **per-response provenance** (one source per call) sits *once* at the envelope level — mixing them makes both harder to consume.

As a proposed EBI default:

```jsonc
{
  "status": "ok",                          // discriminator: "ok" | "not_found" | "partial"
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

Which keys are **mandatory vs optional** should be stated, not left to "should usually contain", so consumers know what they can depend on. Proposed:

| Key | Level | Notes |
|-----|-------|-------|
| `status` | **mandatory** | The success / not-found / partial discriminator. |
| `data` | **mandatory** on success | Array of results; each result carries its own `identifiers`. |
| `identifiers.primary` | **mandatory** per result | The primary identifier of that result. |
| `provenance` | **mandatory** on success | Omitted on error / not-found (see below). |
| `pagination` | mandatory for paginated tools | Include `truncated` and `returned` vs `total` (see [§5.4](#54-result-size-and-pagination)). |
| `identifiers.xrefs` | optional | Cross-references as CURIEs. |

Two refinements from early implementers:

- **Do not attach provenance to errors or "not found".** Those describe the *absence* of data, not data with a source; a provenance block on `{status: "not_found"}` misleads the agent into thinking something was retrieved.
- **Mind the context-cost** of repeating provenance and xrefs on every row. On a long list this can dominate the token budget — a further reason to keep provenance per-response, and to offer a compact projection ([§5.4](#54-result-size-and-pagination)).

### 5.3 Identifiers, provenance and FAIR

- Returns MUST carry the **primary identifier** of each result, and SHOULD carry **cross-references** as resolvable CURIEs, using a prefix from the agreed source (see [§10.1](#101-identifiers)).
- Returns SHOULD include **provenance** on success: the source resource, its data/release version, retrieval timestamp, the **data licence**, and a **citation**. Agents and downstream users must be able to attribute and reproduce. This is the FAIR expectation EBI already holds its APIs to, carried over to MCP. (Provenance is present on success and omitted on error / not-found — see [§5.2](#52-structured-outputs).)
- Numeric values SHOULD state their **units** explicitly — a bare number an agent cannot interpret is a common, silent source of downstream error. Prefer an explicit unit field (ideally a unit CURIE) over embedding the unit in a free-text label.

### 5.4 Result size and pagination

- Tools that can return large result sets MUST paginate (cursor-based RECOMMENDED) and MUST document the default and maximum page size.
- Tools SHOULD **signal truncation explicitly** rather than leaving a caller to infer it: a `truncated` flag and explicit counts (`returned` vs `total`) let an agent tell a silent cut-off from a genuine page boundary and avoid quiet under-counts. This is the companion to pagination, not a substitute for it.
- Tools SHOULD let the caller request a compact projection rather than always returning every field, to keep agent context manageable. This also bounds the per-row context-cost noted in [§5.2](#52-structured-outputs).

---

## 6. Error handling

Consistent error behaviour lets an agent recover (retry, back off, ask the user, try another tool) instead of failing opaquely.

- Distinguish **protocol errors** (malformed request, method not found — surfaced via the JSON-RPC error channel) from **tool-execution errors** (the tool ran but could not complete — surfaced via the result's error flag, e.g. `isError`). Use the right channel for each.
- Tool errors MUST be **structured and actionable**: a stable machine-readable code, a human-readable message, and where possible a hint at remediation ("no record for that accession; check the identifier namespace").
- Errors MUST NOT leak secrets, credentials, internal stack traces, or internal host/network detail.
- **Rate limiting** MUST be signalled explicitly (a distinct, recognisable error condition, with a retry-after indication where available) so agents back off rather than hammer the service. See [§9](#9-authentication-authorisation-and-rate-limiting).
- Where an **upstream** resource returns its own rate-limit response (e.g. HTTP 429), the server SHOULD map it to a **retryable** tool error carrying any `retry-after` hint, distinct from a terminal error — so the agent backs off rather than treating a transitive limit as permanent failure. This is separate from the server's *own* rate limiting (see [§9.3](#93-rate-limiting-and-abuse-handling)).
- Where a long-running operation may exceed client timeouts, servers MAY use the experimental **Tasks** mechanism of the pinned revision (call-now / fetch-later) rather than blocking.
- Partial success SHOULD be representable (return what succeeded, flag what did not) rather than failing the whole call.

---

## 7. Versioning and change management

- Servers MUST version their **software releases** with [Semantic Versioning](https://semver.org/) and maintain a changelog.
- Servers MUST correctly negotiate the **MCP protocol version** with clients and MUST NOT assume a single client version.
- **Breaking changes** to tool names, input schemas or return shapes MUST go through a deprecation cycle: announce, run old and new in parallel for a stated period, then remove. Removing a tool or field without notice breaks every agent built against it.
- Servers SHOULD expose their software version and the MCP revision they target in their server metadata/instructions, so an aggregator and an agent can record what they are talking to.

---

## 8. Discovery and registration

### 8.1 Server instructions and metadata

- Servers SHOULD provide **server instructions** (a short "user manual" for the LLM) describing what the resource is, when to use it, and any house rules.
- Tool, resource and prompt metadata MUST be complete enough that a client can present and select them without out-of-band knowledge.
- Servers MAY additionally publish a **well-known discovery card** (a `/.well-known/mcp`-style document) exposing the server's identity and capability surface, so an aggregator or inventory crawler can read it **without opening a full MCP session**. Early implementers have found this handy for exactly that.

### 8.2 Registration in BioContextAI (Deliverable D2)

EBI MCP endpoints SHOULD be registered for automated discovery. We adopt the **[BioContextAI Registry](https://biocontext.ai/registry)** as the external registry: it is a community catalogue of biomedical MCP servers, it is Schema.org-compliant and downloadable as JSON, and it is the natural home for EBI's public servers. Registration is a pull request adding a `meta.yaml` file (the [online editor](https://biocontext.ai/registry/editor) generates it).

A worked EBI example, conforming to the registry [`schema.json`](https://github.com/biocontext-ai/registry/blob/main/schema.json):

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

To qualify for the BioContextAI Registry a server must, in summary: have a clear **biomedical focus**; be **free for academic use**; carry an **OSI-approved open-source licence**; be **MCP-compliant**; have **public** code and documentation; and not duplicate an existing registry entry without justification. These map cleanly onto our Baseline expectations.

### 8.3 EBI-internal inventory (Deliverable D1)

In parallel with the public registry, the project maintains an **internal living inventory** of EBI resources with current or planned MCP work — endpoints, status (live / prototype / planned), tools exposed, and adoption blockers. The survey at M1 seeds this.

BioContextAI registration requires a **public** repository, which gates registry entry on a visibility decision a team may not control on day one (a server may stay private until a demo). The inventory SHOULD therefore allow a **"registration-ready, pending public"** status, so a server with a schema-validated `meta.yaml` in hand is tracked as ready even before it is submitted. Building the **inventory first** is the priority: the federated-aggregator question below is most useful once there is a handful of servers to aggregate.

**[OPEN]** Do we *also* stand up a lightweight EBI-scoped registry/aggregator (a "meta-MCP" presenting EBI servers under one endpoint), or rely solely on BioContextAI plus the internal inventory? This is the federated-aggregator question and a primary working-group decision — see [§11](#11-open-questions-for-the-working-group).

---

## 9. Authentication, authorisation and rate limiting

> This is the one section we propose as a **hard baseline** for all EBI MCP servers. It will be reviewed with **ITS / Security** before v1.0. Agentic access can bypass assumptions baked into existing web UIs, so the bar must be set explicitly.

### 9.1 Default posture

- Public, read-only, already-open data MAY be exposed **without authentication**, but MUST be **rate-limited** (see §9.3).
- Servers exposing the **HTTP transport** MUST implement **DNS-rebinding protection** by validating the `Host` and `Origin` headers against an allowlist. This currently follows only indirectly from the choice of Streamable HTTP, and an implied requirement is easy to miss in review; it is called out here (and in [§12](#12-conformance-checklist)) as an explicit **Baseline** item so it is caught by review rather than by chance. Closing it is typically a small allowlist.
- A server MUST require authentication for any tool that: **mutates state**; accesses **controlled, embargoed or personal** data; or incurs **significant compute or cost**. As a concrete trigger for that last, open-ended case: treat a tool as "costly" if it **fans out beyond a small number of upstream calls behind a single tool call**, or runs a job **longer than a few seconds** — composite / fan-out tools ([§10.4](#104-server-side-cross-resource-composites)) sit squarely in this zone. The working group should fix the exact thresholds.

### 9.2 How to authenticate (per the pinned revision)

- Where authentication is required, servers MUST act as **OAuth 2.1 Resource Servers** as defined by the specification, including publishing **protected-resource metadata** so clients can discover the authorisation server.
- Clients are required to use **Resource Indicators** ([RFC 8707](https://www.rfc-editor.org/rfc/rfc8707)) so tokens are audience-scoped to one server; servers MUST validate the audience and reject mis-scoped tokens.
- Servers SHOULD support the registration/discovery mechanisms added in the pinned revision (OpenID Connect Discovery; OAuth Client ID Metadata Documents) where they integrate with EBI identity.
- The MCP host is responsible for obtaining **explicit user consent** before invoking tools; servers MUST NOT assume consent and MUST NOT design tools whose safe use depends on the agent hiding what it is doing from the user.

**[OPEN]** Which EBI identity provider/authorisation server do controlled-access EBI MCP servers integrate with, and is there an ELIXIR AAI liaison? To be confirmed (charter notes ELIXIR liaison may be needed at M2).

### 9.3 Rate limiting and abuse handling

- Every server MUST enforce rate limits and MUST signal them as a recognisable, retryable error (see [§6](#6-error-handling)), ideally with a retry-after hint.
- Servers SHOULD document their limits in tool descriptions and server instructions so well-behaved agents stay within them.
- Servers SHOULD have a basic abuse-handling story (per-client throttling, ability to revoke) appropriate to a public service.

### 9.4 Secrets

- Servers MUST NOT log credentials, tokens or personal data. Configuration secrets MUST come from the environment, never the repository.

---

## 10. Cross-resource links and shared vocabularies (Deliverable D4)

The payoff of consistency is that an agent can traverse *several* EBI resources in one workflow. That requires a shared identifier scheme and shared vocabularies.

### 10.1 Identifiers

- Cross-references SHOULD be expressed as **Compact URIs (CURIEs)** with a registered prefix (e.g. `ensembl:ENSG00000139618`, `uniprot:P51587`, `efo:EFO_0000305`), so any consumer can resolve them.
- **[OPEN]** *Which prefix registry is authoritative* is not yet settled. Two candidates are on the table: **[identifiers.org](https://identifiers.org/)** (an EBI resource; narrower and so less ambiguous) and the **[Bioregistry](https://bioregistry.io/)** (broader coverage; a superset in practice). Pinning one avoids two servers resolving the same prefix differently — see [§11.10](#11-open-questions-for-the-working-group).
- Each server MUST document which identifier namespaces it accepts as input and which it returns. Servers SHOULD accept CURIE-form identifiers on input where applicable (see [§5.1](#51-inputs)).

### 10.2 Vocabularies

- Where a tool returns ontology terms (diseases, tissues, assays), it SHOULD use a recognised ontology and return the term **identifier** alongside the label — for example via the **EBI Ontology Lookup Service (OLS)** and ontologies such as **EFO**. This lets agents join across resources on the term ID, not on free-text labels.

### 10.3 Worked example (to be hardened into the D4 reference)

A gene-centric traversal across three EBI / EBI-adjacent resources using one shared identifier:

1. **Ensembl MCP** — `ensembl_lookup_gene(symbol="BRCA2")` → resolves to `ensembl:ENSG00000139618`, and returns an `xrefs` list including `uniprot:P51587`.
2. **Open Targets MCP** — `opentargets_get_associations(target="ensembl:ENSG00000139618")` → disease associations keyed on the *same* Ensembl identifier; diseases returned as `efo:` CURIEs.
3. **UniProt MCP** — `uniprot_get_entry(accession="uniprot:P51587")` → protein-level detail, reached purely by following the cross-reference, with no bespoke glue.

The **Perturbation Catalogue** and the **Ensembl GraphQL proof-of-concept** are the two candidate reference implementations named in the charter; the production D4 example will be built on these.

### 10.4 Server-side cross-resource composites

The traversal in §10.3 assumes the **agent** drives the cross-resource join, one tool call per hop. That is flexible and works well for short chains, but for a hub entity with a long sequential chain it can be unreliable — early implementers report partial joins where the agent mis-orchestrates the sequence.

A complementary pattern is to compose the join **server-side**, behind a single tool call, with a **bounded** fan-out so the upstream services are not overloaded. Both are legitimate; the guidelines name both so a team makes the choice deliberately:

- **Client-side traversal** (§10.3): flexible, and the agent can adapt the path — but it pays N agent turns and can mis-orchestrate a long chain.
- **Server-side composite:** fast and reliable for a known join, and it hides the orchestration — but it couples two resources in one codebase and (per [§9.1](#91-default-posture)) is a "costly" fan-out tool that MUST bound its upstream calls.

A cross-resource composite tool has no natural single prefix; name it per the convention in [§4.1](#41-canonical-format). A bounded server-side composite is a good candidate for a shared recipe.

---

## 11. Open questions for the working group

These are the decisions we are **not** pre-empting in this pre-consultation draft. They are the backbone of the M1 survey and the first meeting.

1. **Federated aggregator.** Do we stand up an EBI-scoped "meta-MCP" that federates EBI servers under one endpoint (aligned with the BioContextAI meta-MCP pattern), or rely on BioContextAI + the internal inventory? (§8)
2. **Tool namespacing.** Prefix convention (`ensembl_…`), server-level namespace, or delegate disambiguation to the aggregator? (§4)
3. **Reference stack.** A single recommended stack (FastMCP + container) for new servers, or language-agnostic? (§3)
4. **Mandatory vs recommended auth bar.** Where exactly is the hard floor, pending ITS/Security review? (§9)
5. **Identity provider.** Which authorisation server for controlled-access EBI servers; is an ELIXIR AAI liaison needed? (§9)
6. **Registry of record.** Is BioContextAI the sole external registry, or do we also register elsewhere? (§8)
7. **Return envelope.** Adopt the proposed envelope (§5.2) as a convention, or lighter-touch?
8. **Benchmark/metric.** The charter calls for a shared performance metric across EBI APIs (D3/M2/M3) — what do we measure, and how?
9. **Tool granularity.** Should the recommended default be **task-shaped tools**, a **generic query tool + curated usage context**, or an explicit *both* (task-shaped tools with a query escape hatch)? Raised by conflicting reviewer experience — task-shaped worked well for one team, a generic query tool out-benchmarked explicit tools for another. (§4.2)
10. **CURIE prefix registry.** Is the authoritative prefix source **identifiers.org** (EBI; narrower, less ambiguous) or the **Bioregistry** (broader coverage)? (§10.1)

---

## 12. Conformance checklist

A server team can self-assess against this. It combines the BioContextAI submission requirements with the EBI-specific items above. **B** = Baseline, **R** = Recommended, **A** = Aspirational.

**Identity, packaging, deployment**
- [ ] (B) Stable, human-readable server name and `owner/repository` identifier
- [ ] (R) Streamable HTTP (remote) / stdio (local); no removed transports
- [ ] (R) One-command install (`uvx …`) and/or container image
- [ ] (R) `mcp.json` client snippet in the README
- [ ] (B) No hard-coded configuration or secrets

**Tools and schemas**
- [ ] (B) Tool names conform to the spec's canonical format
- [ ] (R) EBI namespacing + `verb_object` shape *(pending §11.2)*
- [ ] (B) Every tool has input JSON Schema with parameter descriptions
- [ ] (B) Every tool has a clear description incl. limits
- [ ] (R) Identifier-validation gate before identifier-taking tools (resolve, don't just accept)
- [ ] (R) Structured (machine-readable) outputs, not prose-only
- [ ] (R) Consistent return envelope, with a success / not-found discriminator
- [ ] (B) Primary identifier on every result; (R) cross-refs as CURIEs
- [ ] (R) Provenance on success (source, version, timestamp, licence, citation); omitted on error / not-found
- [ ] (R) Pagination + documented page limits; truncation signalled explicitly (`truncated`, seen vs returned)

**Errors and versioning**
- [ ] (B) Protocol vs tool errors on the correct channel
- [ ] (B) Structured, actionable, secret-free error payloads
- [ ] (B) Rate limiting signalled as a retryable error
- [ ] (R) Upstream 429 relayed as a retryable error with retry-after
- [ ] (B) Semantic Versioning + changelog
- [ ] (B) Correct MCP protocol-version negotiation
- [ ] (R) Deprecation cycle for breaking changes

**Security (Baseline — ITS/Security reviewed)**
- [ ] (B) Auth required for mutation / controlled data / costly ops
- [ ] (B) DNS-rebinding protection (Host/Origin allowlist) on HTTP transports
- [ ] (B) OAuth 2.1 Resource Server posture where auth applies
- [ ] (B) Audience-scoped tokens validated (RFC 8707)
- [ ] (B) Rate limiting enforced even when unauthenticated
- [ ] (B) No credentials/PII in logs

**Discovery, cross-resource, FAIR**
- [ ] (R) Server instructions provided
- [ ] (A) Well-known discovery card (`/.well-known/mcp`-style) for crawlers
- [ ] (R) Registered in BioContextAI with valid `meta.yaml`
- [ ] (B) Documented accepted/returned identifier namespaces
- [ ] (R) Ontology terms returned with IDs (OLS/EFO etc.)
- [ ] (B) OSI-approved open-source licence
- [ ] (B) Public repository and documentation
- [ ] (R) `CITATION.cff`; (R) usage examples; (R) test suite
- [ ] (B) British English; published openly on GitHub from the outset

---

## 13. References

- **MCP specification, revision 2025-11-25** — https://modelcontextprotocol.io/specification/2025-11-25
- **MCP 2025-11-25 changelog** (incl. SEP-986 tool names, OAuth/CIMD, Tasks) — https://modelcontextprotocol.io/specification/2025-11-25/changelog
- **MCP versioning / revisions** — https://modelcontextprotocol.io/specification
- **MCP security best practices** (incl. Host/Origin validation against DNS rebinding) — https://modelcontextprotocol.io/specification/2025-11-25/basic/security_best_practices
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
