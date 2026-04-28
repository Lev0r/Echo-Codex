---
title: Evaluate Graphify (and similar code-graph tools) when app/ exists
summary: Defer adopting code-graph tooling until there is real code to graph. Hand-curated INDEX.md plus frontmatter is sufficient for the docs-only phase.
status: open
tags: [tooling, deferred, memory, navigation]
related:
  - docs/INDEX.md
  - docs/notes/README.md
  - AGENTS.md
created: 2026-04-28
---

# Evaluate Graphify (and similar code-graph tools) when `app/` exists

## Context

We considered Graphify and similar AI-powered code-graphing tools as part of the LLM memory architecture. The motivation is real: a graph view of files, symbols, and dependencies makes large codebases tractable for both humans and agents.

However, Echo Codex is currently pre-Phase-0. There is no `app/`, no `packtools/`, no `packs/`. Our entire knowledge base is markdown under `docs/` plus `AGENTS.md` and the rule files under `.cursor/rules/`. The Graphify sweet spot — auto-graphing code symbols and dependencies — does not apply yet.

For docs-only navigation we settled on:

- A hand-curated [`docs/INDEX.md`](../INDEX.md) as the navigation surface.
- YAML frontmatter on every doc (`title`, `summary`, `tags`, `related`, `status`) so future tooling can regenerate the index mechanically when volume justifies it.
- An always-on Cursor rule ([`.cursor/rules/docs-map.mdc`](../../.cursor/rules/docs-map.mdc)) that injects a compact map at session start.
- The existing `.obsidian/` vault, which already gives humans a graph view from frontmatter and wikilinks for free.

## Status

**Open — deferred.** No tooling decision yet; no dependency added.

## Revisit trigger

Reconsider when **any** of the following becomes true:

- `app/` is scaffolded (Phase 0 complete) and has more than ~50 source files.
- Doc count under `docs/` exceeds ~30 files and the hand-curated `INDEX.md` shows signs of drift.
- A concrete agent task is repeatedly blocked by inability to navigate the codebase.

At that point: spike Graphify (and one or two alternatives — e.g. Sourcegraph, ast-grep + a custom indexer, or a simple frontmatter-driven generator) on the then-current repo. If adopted, write an ADR per [`AGENTS.md`](../../AGENTS.md) §6.

## Why not now

- No code to graph; benefit is theoretical.
- Adds a dependency and a build step before we have a build system.
- Risk of locking into a tool whose value-shape doesn't match what we actually need by the time we have code.
- The cheap alternative (frontmatter + hand-curated index + Obsidian) is sufficient for current scale and produces structured metadata that any future tool can ingest.

## References

- [`docs/INDEX.md`](../INDEX.md) — current navigation surface.
- [`docs/notes/README.md`](./README.md) — the policy this note follows.
- The plan that introduced this note: `LLM memory and rules architecture` (2026-04-28).
