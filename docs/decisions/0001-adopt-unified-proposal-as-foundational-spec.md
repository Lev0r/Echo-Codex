---
title: "ADR-0001: Adopt the Unified Proposal as the foundational spec"
summary: Adopts docs/spec.md as the single source of truth and moves predecessor docs to archive/.
tags: [governance, spec, adr]
related:
  - docs/spec.md
status: accepted
---

# ADR-0001: Adopt the Unified Proposal as the foundational spec

- **Status:** Accepted
- **Date:** 2026-04-23
- **Deciders:** Project founder
- **Supersedes:** —

## Context

The project began with two overlapping but conflicting vision documents — *Echo Codex project overview* and *Eco DND Project Specification* — and a proof-of-concept Python scraper. Those documents disagreed on storage (YAML files vs SQLite), data entry (community scrapers vs manual templates), and scope (cloud ecosystem vs self-contained PWA). Leaving both as "sources of truth" would have made every subsequent decision ambiguous.

Over several iterations with an AI agent, the two documents were reconciled and refined into a single coherent design, addressing:

- Real-world operating constraints (free hosting, mobile PWA delivery).
- Legal separation between our own schemas and third-party game content.
- A system-agnostic template-driven rendering engine.
- A deliberately data-driven UI with gesture-first navigation.
- A monetisation strategy compatible with the open-source ethos.

## Decision

[`docs/spec.md`](../spec.md) — "Echo Codex — Unified Proposal" — is the **single source of truth** for the project's design, architecture, UX, roadmap, licensing, and monetisation strategy. All subsequent decisions must either align with it or explicitly supersede parts of it via a new ADR.

The two original documents and the Python scraper have been moved to `archive/`, which is excluded from AI indexing via `.cursorignore` and from the IDE via the workspace file. They remain in git history for provenance.

## Consequences

**Positive.**

- One anchor for every future discussion; no more ambiguity about "which doc is right."
- Clear legal and architectural separation between template packs (ours) and data packs (third-party), enabling the app to remain free of IP risk.
- A concrete phase-by-phase roadmap (Phase 0 → Phase 7) that is ready to execute.
- The spec's §12 captures decisions already made, reducing re-litigation.

**Negative / neutral.**

- The spec is long (~750 lines); contributors must be willing to read the relevant section before acting.
- Changes to the spec require discipline (update §13, add a superseding ADR for meaningful shifts).
- Some topics (derived fields, paste-assist, some renderer details) are deliberately deferred. Teams expecting everything to be settled may find this uncomfortable; it is intentional.

## Alternatives considered

1. **Keep both original documents as joint sources.** Rejected: they disagree on load-bearing choices (storage, data entry, hosting). Every downstream decision would require reconciling them again.
2. **Write a minimal one-pager and defer detail.** Rejected: the conflicting decisions in the source docs required concrete resolution, not deferral. A one-pager would have hidden the unresolved questions.
3. **Pick one of the two originals and discard the other.** Rejected: each contributed useful ideas the other lacked (the renderer-engine / swipe-UX model from doc 1; the PWA / SQLite / counters model from doc 2).

## References

- [`docs/spec.md`](../spec.md) — the adopted spec.
- `archive/Echo-Codex project overview.md` — predecessor (Echo Codex).
- `archive/Eco DND Project Specification.md` — predecessor (Eco DND).
- `archive/spells_scraper.py` — early proof-of-concept scraper; concepts fold into Phase 4a pack tooling.
