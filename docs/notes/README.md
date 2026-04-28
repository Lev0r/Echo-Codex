---
title: Notes
summary: Policy for the agent-writable notes layer. Captures open questions, exploration logs, and session summaries that are explicitly not load-bearing.
tags: [meta, memory, process]
related:
  - docs/INDEX.md
  - docs/decisions/README.md
  - AGENTS.md
status: living
---

# Notes

Agent-writable memory. This folder is where humans and AI agents capture in-progress thinking that is not yet (and may never be) a decision, a spec change, or an implementation document.

## What goes here

- **Open questions** that need investigation before they can become decisions.
- **Exploration logs** — "I tried X, it failed because Y, here's what we learned."
- **Session summaries** scoped to a single topic (not a transcript dump).
- **Hypotheses awaiting evidence** — design ideas that aren't ready for an ADR yet.
- **Cross-session context** an agent wants to leave for the next agent picking up the same thread.

## What does NOT go here

- **Decisions** → write an ADR in [`docs/decisions/`](../decisions/). ADRs are immutable once accepted.
- **Implementation specs** → write or extend an architecture doc in [`docs/architecture/`](../architecture/).
- **How-tos** → write a guide in [`docs/guides/`](../guides/).
- **Spec changes** → edit [`docs/spec.md`](../spec.md) and add an ADR per [`AGENTS.md`](../../AGENTS.md) §5.
- **Session transcripts or chat logs.** A note distils insight; it isn't a paste of conversation.

If a note grows past a page, it's probably ready to promote — see "Promotion paths" in [`docs/INDEX.md`](../INDEX.md).

## Status of this layer

Notes are explicitly **not load-bearing.** Agents and humans should consult them for prior context, but treat them as observations, not commitments. When a note's content disagrees with `spec.md`, an ADR, or an architecture doc, the latter wins. When a note's status is `archived`, ignore it.

## File naming

Two acceptable patterns:

- **Topical:** `topic-slug.md` — preferred when the note is about a specific subsystem or question. Example: `pack-loading-cache-strategy.md`.
- **Dated:** `YYYY-MM-DD-short-slug.md` — preferred for session summaries or when multiple notes on the same topic accumulate. Example: `2026-04-28-renderer-spike.md`.

Use kebab-case. Keep slugs short.

## Required frontmatter

Every note file must start with frontmatter:

```yaml
---
title: <human-readable title>
summary: <one sentence, ~140 chars>
status: open | resolved | archived
tags: [renderer, packs, sync, ...]
related:
  - docs/spec.md
  - docs/decisions/0001-....md
created: YYYY-MM-DD
updated: YYYY-MM-DD   # optional; set when materially edited
---
```

Status semantics:

- **`open`** — actively relevant; consult when working on the related area.
- **`resolved`** — the question was answered. Body should link to the resolving ADR / architecture doc / spec section. Kept for audit trail; can be archived later.
- **`archived`** — superseded, stale, or no longer relevant. Agents should skip these unless explicitly asked.

## Promotion and pruning

When a note matures, **promote rather than expand**:

- Open question becomes a real decision → write an ADR; update the note's status to `resolved` and link to the ADR.
- Exploration becomes a subsystem design → write an [`architecture/`](../architecture/) doc; mark the note `resolved` and link.
- Note is no longer relevant → set status to `archived`. Do not delete; provenance matters.

Run a sweep at the end of each project phase (or quarterly, whichever is sooner): promote, archive, or delete obviously dead notes. Stale notes in a memory layer are worse than no notes — they mislead future agents.

## Indexing

Add every new note to the **Notes** section of [`docs/INDEX.md`](../INDEX.md). The one-liner there should match the note's frontmatter `summary:`.
