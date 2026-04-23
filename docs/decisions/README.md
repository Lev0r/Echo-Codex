# Architecture Decision Records

Architecture Decision Records (ADRs) capture significant choices that shape the project. They exist so future contributors (and our future selves) can understand *why* the project looks the way it does — not just *what* the code does.

## When to write one

- Introducing a new third-party dependency of any size.
- Changing a non-negotiable listed in [`../../AGENTS.md`](../../AGENTS.md) §2.
- Choosing between materially different technical approaches (e.g. "SQLite vs IndexedDB").
- Adding a new pack type, file format, or storage layer.
- Significant scope changes (moving a feature between MVP / V1.1 / V2).

Don't write one for routine implementation work, bug fixes, or small refactors.

## Protocol

- **Numbering:** zero-padded, monotonic, never reused. `0001-`, `0002-`, …
- **Filename:** `NNNN-short-kebab-slug.md`.
- **Statuses:** `Proposed`, `Accepted`, `Rejected`, `Superseded by ADR-NNNN`, `Deprecated`.
- **Immutable once `Accepted`.** Do not rewrite accepted ADRs; if the decision changes, write a new ADR and mark the old one `Superseded by ADR-NNNN`.
- **Keep them short.** Context + decision + consequences. If you need more than a page, something is wrong with the scope.

## Template

```markdown
# ADR-NNNN: <short title, imperative mood>

- **Status:** Proposed | Accepted | Rejected | Superseded by ADR-NNNN | Deprecated
- **Date:** YYYY-MM-DD
- **Deciders:** <names or handles>
- **Supersedes:** ADR-NNNN (optional)

## Context

What problem are we solving? What constraints apply? What does the spec say about the area touched?

## Decision

What we are going to do, stated plainly. One or two paragraphs.

## Consequences

Positive, negative, and neutral follow-ons. What becomes easier, what becomes harder, what we now must not do.

## Alternatives considered

Briefly, with the reason each was rejected.

## References

Relevant spec sections, issues, prior ADRs, external docs.
```

## Log

| ADR  | Title                                       | Status   | Date       |
| ---- | ------------------------------------------- | -------- | ---------- |
| [0001](./0001-adopt-unified-proposal-as-foundational-spec.md) | Adopt the Unified Proposal as the foundational spec | Accepted | 2026-04-23 |
