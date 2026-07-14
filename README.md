# EMBL-EBI MCP Guidelines

Shared, EBI service-wide guidelines for building **Model Context Protocol (MCP)** servers, so that agentic access to EBI resources is consistent and interoperable in tool naming, schemas, authentication, error handling, discovery and cross-resource links.

This repository is **public from day one** for external visibility. It is maintained by the *Common Guidelines on MCPs* working group at EMBL-EBI.

## Status

| | |
|---|---|
| **Current document** | [`GUIDELINES.md`](./GUIDELINES.md) |
| **Version** | v0.2 — pre-consultation draft (strawman + first-round feedback incorporated) |
| **Pinned MCP revision** | [`2025-11-25`](https://modelcontextprotocol.io/specification/2025-11-25) (current stable) |
| **Roadmap** | v0.5 working draft (M3) → v0.9 release candidate (M4) → **v1.0** published (M5) |

v0.2 stays deliberately opinionated so reviewers have something concrete to react to, and folds in the first round of reviewer feedback (June–July 2026). Nothing is ratified yet; newly surfaced disagreements — tool granularity, and which CURIE prefix registry is authoritative — are captured as additional [Open questions](./GUIDELINES.md#11-open-questions-for-the-working-group) rather than resolved unilaterally.

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
