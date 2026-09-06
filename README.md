# EMBL-EBI MCP Guidelines

Shared guidelines for building **Model Context Protocol (MCP)** servers at EMBL-EBI. They help teams make their data and tools available to AI applications using consistent names, formats and security practices.

This repository is **public from day one** for external visibility. It is maintained by the *Common Guidelines on MCPs* working group at EMBL-EBI.

## Status

| | |
|---|---|
| **Current document** | [`GUIDELINES.md`](./GUIDELINES.md) |
| **Version** | v0.3 — pre-consultation draft (MCP 2026-07-28 baseline) |
| **Last revised** | 6 September 2026 |
| **Pinned MCP revision** | [`2026-07-28`](https://modelcontextprotocol.io/specification/2026-07-28) (current stable) |
| **Roadmap** | v0.5 working draft (M3) → v0.9 release candidate (M4) → **v1.0** published (M5) |

v0.3 includes early reviewer feedback and uses MCP `2026-07-28`. The 6 September revision corrects requirement levels, clarifies migration and simplifies the language. The working group has not approved the draft yet. Choices about tool design, identifier prefixes and registries remain [open questions](./GUIDELINES.md#11-open-questions-for-the-working-group).

For a first read, start with the [scope](./GUIDELINES.md#1-purpose-and-scope), [open questions](./GUIDELINES.md#11-open-questions-for-the-working-group) and [checklist](./GUIDELINES.md#12-conformance-checklist). If upgrading an older MCP implementation, read [§2.4 Migrating from an earlier revision](./GUIDELINES.md#24-migrating-from-an-earlier-revision).

## What this covers

The guidelines cover server identity and deployment, tool names and design, input and output formats, errors, versioning, discovery and registration, security, rate limits, and identifiers shared across resources.

What it does **not** cover: building or hosting servers for other teams, changing the MCP protocol upstream, or delivering a cross-resource agent product. These are out of scope or proposed follow-on work.

## How to contribute

- **Comment on the draft:** open an issue, or a pull-request suggestion against `GUIDELINES.md`.
- **Add a server:** this draft uses the [BioContextAI Registry](https://github.com/biocontext-ai/registry) for public entries. The working group also maintains an internal inventory of current and planned servers, starting with the M1 survey responses.
- All written outputs use **British English**.

## Project

- **Co-sponsors:** Fergal Martin, Henning Hermjakob
- **Coordinator:** Alexey Sokolov
- **Portfolio:** EMBL-EBI AI Pilots
- Fortnightly working-group meetings; monthly updates to sponsors and the AI Pilots portfolio.

## Licence

To be confirmed by the working group — an OSI-approved open-source licence (the guidance itself will likely sit under CC BY 4.0, with any code under Apache-2.0). See [Open questions](./GUIDELINES.md#11-open-questions-for-the-working-group).
