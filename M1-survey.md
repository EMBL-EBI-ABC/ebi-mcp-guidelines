# Common Guidelines on MCPs — M1 Stakeholder Survey

**Version 0.2 (feedback incorporated; draft for sponsor review)** · Project: *Common Guidelines on MCPs*, EMBL-EBI

This is a **tool-agnostic specification** of the survey: question text, type, answer options and routing. Paste it into your survey tool of choice (EUSurvey, Google Forms, Microsoft Forms) — the structure is preserved.

It does two jobs at once:

1. **Seeds the D1 inventory** of EBI resources with current or planned MCP work.
2. **Gathers steer on the open design questions** from the v0.2 guidelines (`GUIDELINES.md §11`), so consolidation at M2 maps one-to-one onto the decisions the working group needs to make.

Branching keeps it short: most respondents answer ~14–17 questions (≈10 minutes). Only teams already building or planning a server see the full inventory section.

> Internal mapping (questions → deliverables, deployment notes, and a draft invitation) is at the end under **[Notes for the working group](#notes-for-the-working-group-not-shown-to-respondents)** — not shown to respondents.

---

## Respondent-facing introduction

> **EMBL-EBI MCP Guidelines — your input (≈10 minutes)**
>
> EBI service teams are increasingly asked to expose data and tools to LLM-based agents via the **Model Context Protocol (MCP)**. Several teams are already building MCP servers; demand from consortia and industry is growing. Without shared conventions, every resource invents its own approach to naming, schemas, authentication and discovery, which makes cross-resource workflows brittle.
>
> This working group (co-sponsored by **Fergal Martin** and **Henning Hermjakob**) is producing EBI service-wide guidelines for building MCP servers. This survey tells us **who is doing what** and **what the guidelines must get right for you**.
>
> A v0.2 strawman draft is already public: *[link to GUIDELINES.md]*. You do **not** need to read it to respond — but if you do, there's space below to disagree with it.
>
> Your contact details are used only for the resource inventory and any follow-up, and to invite you to a first meeting if you'd like. All outputs are published openly on GitHub in British English.
>
> **Please complete the survey once for every resource you represent.** As a guide to granularity, use the EBI services list as defined by [confirm the authoritative list with Matt Thakur / Simone Vegh].
>
> **Please respond by [date — suggest two weeks out].**

---

## Section 1 — You and your resource

**Q1. Your name** *(short text, optional)*

**Q2. Email** *(short text — used only for the inventory and follow-up)*

**Q3. Which EBI resource or team do you represent?** *(short text)*
*Examples: Ensembl, UniProt, PDBe, Europe PMC, Open Targets, Expression Atlas…*

**Q4. Your role** *(short text)*

**Q5. What is the status of MCP work for your resource?** *(single choice — routing question)*
- Live — we run an MCP server in production
- Prototype / in active development
- Planned within the next 12 months
- Considering it, no timeline yet
- No current plans
- Not sure

> **Routing:** *Live / Prototype / Planned* → **Section 2** (inventory). All other answers → skip to **Section 3**.

---

## Section 2 — Your MCP server *(shown only for Live / Prototype / Planned)*

*Answer what you can; "don't know / not yet decided" is a valid answer throughout.*

**Q6. Endpoint URL of the MCP server, if hosted** *(short text)*

**Q7. Source-code repository URL** *(short text)*

**Q8. Transport** *(single choice)*
- Streamable HTTP
- stdio (local)
- HTTP + SSE (legacy)
- Other
- Don't know

**Q9. What does it expose? Roughly how many tools, and what do they do?** *(long text)*

**Q10. Which data and tooling does the server make available?** *(long text)*

**Q11. Implementation language / framework** *(single choice)*
- Python + FastMCP
- Python + other framework
- TypeScript / JavaScript
- Other
- Not yet decided

**Q12. Authentication** *(single choice)*
- None (open, read-only)
- API key
- OAuth 2.x
- Other
- Not yet decided

**Q13. Where is it (or will it be) hosted?** *(short text — e.g. EBI infrastructure, cloud provider, undecided)*

**Q14. Is it registered for discovery anywhere?** *(single choice)*
- Yes — BioContextAI Registry
- Yes — elsewhere *(please name)*
- No
- Don't know

**Q15. What are the biggest blockers to building or adopting an MCP server for your resource?** *(checkboxes — select all that apply)*
- Time / capacity
- In-house MCP expertise
- Security and authentication uncertainty
- Hosting / infrastructure
- Absence of shared standards
- Unclear demand from users
- Funding
- Other *(please specify)*

---

## Section 3 — Design questions *(shown to everyone)*

*These steer the guidelines directly. Every question accepts **"No view"** — click through quickly if a topic isn't yours.*

**Q16. Should EBI provide a single federated endpoint that aggregates EBI MCP servers (a "meta-MCP"), *in addition to* the individual service-level servers, so an agent can reach many resources through one connection?** *(single choice)* — `§11.1`
- Strongly support · Support · Neutral · Oppose · Strongly oppose · No view

**Q17. How should tools be named so they stay unambiguous when many EBI servers are aggregated?** *(single choice)* — `§11.2`
- Resource prefix on every tool (e.g. `ensembl_lookup_gene`)
- Server-level namespace, plain tool names within it
- Leave disambiguation to the aggregator
- No preference / no view

**Q18. For tool design, which should the guidelines recommend as the default?** *(single choice)* — `§11.9`
- A small set of **task-shaped tools** (each wrapping ~one operation)
- A **generic query tool** plus curated usage context and examples
- **Both** — task-shaped tools with a generic query escape hatch
- Depends on the resource
- No preference / no view

**Q19. For new EBI servers, should the guidelines recommend a single reference stack (e.g. FastMCP + container)?** *(single choice)* — `§11.3`
- Yes — recommend one stack
- No — stay language-agnostic
- No view

**Q20. For public, read-only EBI data, authentication should be…** *(single choice)* — `§11.4`
- Not required, but rate-limited
- Required for all access
- Depends on the resource
- No view

**Q21. Do you support a mandatory minimum security baseline that every EBI MCP server must meet (reviewed with ITS/Security)?** *(single choice)* — `§11.4`
- Yes · Yes, if kept lightweight · No · No view

**Q22. For controlled-access servers, which identity system should they integrate with?** *(single choice)* — `§11.5`
- EMBL/EBI single sign-on
- ELIXIR AAI
- Don't know
- Other *(please specify)*

**Q23. Is the [BioContextAI Registry](https://biocontext.ai/registry) the right external registry for EBI public servers?** *(single choice)* — `§11.6`
- Yes — use it
- Yes, but register elsewhere too *(please name where)*
- Prefer an EBI-only registry
- No view

**Q24. Would a shared return envelope (each result carrying its identifiers, provenance/citation and pagination) be useful to standardise?** *(single choice)* — `§11.7`
- Yes — adopt as a convention
- Lighter-touch guidance is enough
- No view

**Q25. Which prefix registry should be authoritative for the CURIEs EBI servers emit?** *(single choice)* — `§11.10`
- identifiers.org (EBI resource; narrower, less ambiguous)
- Bioregistry (broader coverage; a superset in practice)
- Either / no strong preference
- No view

**Q26. To know an EBI API performs *well* under agentic access, what would you want measured?** *(long text — optional)* — `§11.8`
*e.g. latency, result correctness, token efficiency, rate-limit headroom…*

**Q27. What is the single most important thing these guidelines must get right for you?** *(long text)*

**Q28. If you've looked at the v0.2 draft — is there anything in it you disagree with?** *(long text, optional)*

---

## Section 4 — Getting involved *(shown to everyone)*

**Q29. Would you like to join the working group?** *(single choice)*
- Yes — active member
- Yes — occasional reviewer of drafts
- Keep me informed only
- No, thank you

**Q30. Would you attend a first kick-off meeting?** *(single choice)*
- Yes · Maybe · No

**Q31. Could your server be a candidate reference implementation for the guidelines?** *(single choice)*
- Yes · Maybe · No · Not applicable

**Q32. Anything else you'd like the working group to know?** *(long text, optional)*

> *Thank you. Findings will be consolidated and shared back with the **MCP guidelines working group** (and with respondents), and the draft guidelines updated openly on GitHub — consistent with the open-publication note above.*

---
---

## Notes for the working group *(not shown to respondents)*

### Question → deliverable map

| Questions | Feeds |
|-----------|-------|
| Q1–Q4, Q6–Q14 | **D1** internal inventory (endpoints, status, tooling, hosting, registration) |
| Q5 | Inventory status field + routing |
| Q15 | Adoption blockers (D1; informs the support follow-on programme) |
| Q16 | `§11.1` federated aggregator |
| Q17 | `§11.2` tool namespacing |
| Q18 | `§11.9` tool granularity |
| Q19 | `§11.3` reference stack |
| Q20, Q21 | `§11.4` auth bar / mandatory baseline (→ ITS/Security agenda) |
| Q22 | `§11.5` identity provider (→ surfaces ELIXIR AAI liaison, charter M2 assumption) |
| Q23 | `§11.6` registry of record |
| Q24 | `§11.7` return envelope |
| Q25 | `§11.10` CURIE prefix registry |
| Q26 | `§11.8` benchmark/metric (→ M2 performance-metric specification) |
| Q27, Q28 | Qualitative steer + draft critique |
| Q29–Q31 | Working-group membership, meeting list, reference-implementation candidates |
| Q32 | Open comments |

At **M2**, each closed design question (Q16–Q26) consolidates into a headline distribution ("support a federated endpoint: 70% / oppose: 10% / no view: 20%") that you take to the sponsor checkpoint and fold into the v0.5 draft. The mapping above means consolidation is mechanical, not interpretive.

### Deployment notes

- **Tool:** EUSurvey (EC-hosted, common at EMBL-EBI) or Google Forms both work; Q5's routing and Q15's checkboxes are supported by both. Keep Section 2 on its own page gated by Q5.
- **Not fully anonymous by design** — the inventory needs to know which resource each answer is about. The intro is explicit that contact details are used only for the inventory and follow-up; that's the honest trade and worth keeping.
- **Distribution:** GTL/resource-lead mailing lists first; confirm with **Simone Vegh** the channel for reaching non-GTL resource staff (open charter question). Co-sponsor names in the intro are doing the engagement work — keep them prominent given low-engagement is a top risk.
- **Window:** ~2 weeks, one reminder at the one-week mark; surface non-respondents at the sponsor checkpoint.
- **Privacy:** if collecting names/emails, a one-line data-handling note (purpose, retention, who sees it) keeps it clean under EMBL data policy.

### Draft invitation (email / Slack — tailor before sending)

> **Subject: 10 minutes? Shaping EBI's MCP guidelines — your resource**
>
> Hi [name],
>
> EBI is increasingly asked to expose data and tools to AI agents over the Model Context Protocol. To stop every team reinventing naming, schemas, auth and discovery, Fergal Martin, Henning Hermjakob and I are drafting shared EBI guidelines for building MCP servers.
>
> Could you spare ~10 minutes for a short survey? It tells us whether your resource has (or plans) an MCP server, and what the guidelines must get right for you. A public strawman draft is here if you're curious: [link].
>
> Survey: [link] — by [date].
>
> If you'd rather just talk it through, I'm very happy to. There'll be a kick-off meeting once responses are in.
>
> Thanks,
> Alexey
