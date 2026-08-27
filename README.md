# EMBL-EBI MCP Guidelines

Shared, EBI service-wide guidelines for building **Model Context Protocol (MCP)** servers, so that agentic access to EBI resources is consistent and interoperable in tool naming, schemas, authentication, error handling, discovery and cross-resource links.

This repository is **public from day one** for external visibility. It is maintained by the *Common Guidelines on MCPs* working group at EMBL-EBI.

## Status

| | |
|---|---|
| **Current document** | [`GUIDELINES.md`](./GUIDELINES.md) |
| **Version** | v0.3 — pre-consultation draft (MCP 2026-07-28 baseline) |
| **Pinned MCP revision** | [`2026-07-28`](https://modelcontextprotocol.io/specification/2026-07-28) (current stable) |
| **Roadmap** | v0.5 working draft (M3) → v0.9 release candidate (M4) → **v1.0** published (M5) |

v0.3 stays deliberately opinionated so reviewers have something concrete to react to. It retains the first-round feedback incorporated in v0.2 and updates the protocol baseline to the final MCP `2026-07-28` revision. Nothing is ratified yet; disagreements — including tool granularity, the CURIE prefix registry, and registry-of-record choices — remain [Open questions](./GUIDELINES.md#11-open-questions-for-the-working-group) rather than being resolved unilaterally.

## What this covers

Server identity and deployment · tool naming and granularity · input and return schemas · error handling · versioning · discovery and registration (incl. the [BioContextAI Registry](https://biocontext.ai/registry)) · an authentication / authorisation / rate-limiting **security baseline** · and cross-resource identifier and vocabulary conventions.

What it does **not** cover: building or hosting servers for other teams, changing the MCP protocol upstream, or delivering a cross-resource agent product. These are out of scope or proposed follow-on work.

## How to contribute

- **Comment on the draft:** open an issue, or a pull-request suggestion against `GUIDELINES.md`.
- **Add a server to the picture:** the public catalogue of record is the [BioContextAI Registry](https://github.com/biocontext-ai/registry); the EBI-internal inventory of current/planned servers is maintained by the working group (seeded by the M1 survey).
- All written outputs use **British English**.

## Project

- **Co-sponsors:** Fergal Martin, Henning Hermjakob
- **Coordinator:** Alexey Sokolov
- **Portfolio:** EMBL-EBI AI Pilots
- Fortnightly working-group meetings; monthly updates to sponsors and the AI Pilots portfolio.

## Licence

To be confirmed by the working group — an OSI-approved open-source licence (the guidance itself will likely sit under CC BY 4.0, with any code under Apache-2.0). See [Open questions](./GUIDELINES.md#11-open-questions-for-the-working-group).
