---
title: Documentation index
summary: Canonical, hand-curated map of every doc in the repository, organised by intent. The navigation surface for humans and agents.
tags: [meta, index, navigation]
related:
  - docs/README.md
  - AGENTS.md
status: living
---

# Documentation index

Hand-curated map of every doc in the repository, grouped by intent. Read [`AGENTS.md`](../AGENTS.md) first for non-negotiables; come here for navigation.

For folder conventions and editing policy see [`docs/README.md`](./README.md). For the compressed map injected into every Cursor session see [`.cursor/rules/docs-map.mdc`](../.cursor/rules/docs-map.mdc).

## Reading order for a new agent

1. [`AGENTS.md`](../AGENTS.md) — non-negotiables, repo map, conventions.
2. [`docs/spec.md`](./spec.md) §1–2 — what the product is and the design principles.
3. This file (`docs/INDEX.md`) — pick the next thing to read based on the task at hand.
4. [`docs/decisions/`](./decisions/) — ADR log, in case the area you're touching has a prior decision.
5. [`docs/notes/`](./notes/) — open questions and prior exploration relevant to the task.

## Where to look by task type

- **Designing or scoping a feature** → [`docs/spec.md`](./spec.md). Search for the matching numbered section first.
- **Choosing between approaches** → [`docs/decisions/README.md`](./decisions/README.md) (template + when to write an ADR), then existing ADRs.
- **Implementing a subsystem** → [`docs/architecture/`](./architecture/) (currently empty; plan-of-record is in `spec.md` §6).
- **Writing or running something concrete** → [`docs/guides/`](./guides/) (currently empty; anticipated entries listed in its README).
- **Capturing in-progress thinking, open questions, or session-end summaries** → [`docs/notes/`](./notes/).
- **Pack authoring / pack rules** → [`docs/spec.md`](./spec.md) §5 and §6.3, plus [`.cursor/rules/packs.mdc`](../.cursor/rules/packs.mdc).
- **Agent conventions** → [`AGENTS.md`](../AGENTS.md), with thin shims in [`.cursor/rules/`](../.cursor/rules/), `CLAUDE.md`, and `.github/copilot-instructions.md`.

## Design

- [`docs/spec.md`](./spec.md) — *living* — the unified design specification; single source of truth for architecture, UX, roadmap, licensing, monetization. Sections: 1 Executive summary, 2 Design principles, 3 Product scope, 4 System-agnosticism, 5 Information architecture, 6 Technical architecture, 7 UX/UI, 8 Localization, 9 Licensing and monetization, 10 Cost model, 11 Roadmap, 12 Decisions and open questions, 13 Change log. Tags: `design`, `canonical`.

## Decisions (ADRs)

- [`docs/decisions/README.md`](./decisions/README.md) — *living* — when to write an ADR, numbering, statuses, template, and the running log. Tags: `meta`, `process`.
- [`docs/decisions/0001-adopt-unified-proposal-as-foundational-spec.md`](./decisions/0001-adopt-unified-proposal-as-foundational-spec.md) — *accepted* — adopts `docs/spec.md` as the single source of truth and moves the predecessor docs to `archive/`. Tags: `governance`, `spec`.

## Architecture (living technical docs)

- [`docs/architecture/README.md`](./architecture/README.md) — *living* — index and conventions for living technical docs. Anticipated entries: `data-model.md`, `rendering-engine.md`, `navigation.md`, `pack-loading.md`, `sync-protocol.md`, `performance-budget.md`. Currently no concrete docs — see `spec.md` §6 for the plan of record. Tags: `meta`, `architecture`.

## Guides (task-oriented how-tos)

- [`docs/guides/README.md`](./guides/README.md) — *living* — index and conventions for guides. Anticipated entries: `dev-setup.md`, `pack-authoring.md`, `pack-tooling.md`, `release-process.md`, `contributing.md`. Currently no concrete guides — the codebase doesn't exist yet. Tags: `meta`, `how-to`.

## Notes (agent-writable memory)

- [`docs/notes/README.md`](./notes/README.md) — *living* — policy for the agent-writable notes layer: what belongs here vs. ADRs / architecture / guides, frontmatter requirements, status lifecycle, and promotion paths. Tags: `meta`, `memory`.
- [`docs/notes/graphify-evaluation.md`](./notes/graphify-evaluation.md) — *open* — deferred evaluation of Graphify and similar code-graph tooling; revisit when `app/` scaffolding lands. Tags: `tooling`, `deferred`.

## Agent rules (one canonical source, many shims)

- [`AGENTS.md`](../AGENTS.md) — *canonical* — the single source of truth for agent and contributor behaviour: project summary, non-negotiables, repo map, documentation discipline, ADR protocol, coding conventions, commit/branch/PR rules, memory layers.
- [`.cursor/rules/core.mdc`](../.cursor/rules/core.mdc) — *shim* — Cursor always-on core rule; restates the non-negotiables in one line each and links to `AGENTS.md`.
- [`.cursor/rules/docs-map.mdc`](../.cursor/rules/docs-map.mdc) — *shim* — Cursor always-on bootstrap; injects a compact doc map and "where to look" cheatsheet pointing here.
- [`.cursor/rules/documentation.mdc`](../.cursor/rules/documentation.mdc) — *shim* — Cursor rule applied when editing docs; conventions, frontmatter, notes layer.
- [`.cursor/rules/packs.mdc`](../.cursor/rules/packs.mdc) — *shim* — Cursor rule applied when editing pack YAML; pack authoring rules.
- [`CLAUDE.md`](../CLAUDE.md) — *shim* — redirects Claude Code to `AGENTS.md`.
- [`.github/copilot-instructions.md`](../.github/copilot-instructions.md) — *shim* — redirects GitHub Copilot to `AGENTS.md`.

## Promotion paths

Content moves between layers as it matures. The expected promotions:

- **`notes/` → `decisions/`** — when an open question becomes a deliberate decision, write an ADR. The note can link to the ADR and update its `status` to `resolved`, or be archived.
- **`notes/` → `architecture/`** — when exploration crystallises into a concrete subsystem design, write an architecture doc. Link from the note; archive the note when superseded.
- **`notes/` → `spec.md`** — only via an ADR. The spec changes only when a meaningful decision has been recorded.
- **`architecture/` → `spec.md`** — rare; only when an architecture detail has become a load-bearing design commitment. Requires an ADR.
- **`decisions/`** — never demoted. Superseded ADRs stay; a new ADR records the change.
- **`spec.md`** — never demoted. Sections deprecate via §13 ("What changed vs. the source documents") and ADRs.

The `archive/` folder is **not** a promotion target. It holds historical, superseded materials and is excluded from AI indexing.

## Maintenance

- This file is hand-curated. When adding a new doc, add an entry here in the matching section.
- Each entry's one-liner should match the doc's frontmatter `summary:` field — drift is a smell.
- When the doc count grows or frontmatter coverage is universal, this index becomes a candidate for mechanical regeneration.
