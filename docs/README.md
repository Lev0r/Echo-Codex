# Echo Codex documentation

This folder is the project's living knowledge base. It is maintained jointly by human contributors and AI agents following the conventions in [`../AGENTS.md`](../AGENTS.md).

## Structure

| Path              | Purpose                                                                    | Editing policy                           |
| ----------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| [`spec.md`](./spec.md) | The unified design specification. Single source of truth for architecture, UX, roadmap, licensing, and monetization. | Edit carefully; update §13 on change. Add an ADR for meaningful decisions. |
| [`decisions/`](./decisions/) | Architecture Decision Records. Numbered, immutable once `Accepted`. | Add only; supersede rather than rewrite. |
| [`architecture/`](./architecture/) | Living technical documentation (data model, rendering engine, sync protocol, …). | Rewrite freely.                          |
| [`guides/`](./guides/) | Task-oriented how-tos (dev setup, pack authoring, release process, contributing). | Rewrite freely.                          |

## Reading order for a new contributor

1. [`../README.md`](../README.md) — what the project is in 30 seconds.
2. [`spec.md`](./spec.md) — the full design. Start with §1 and §2, then skim §5 and §9.
3. [`../AGENTS.md`](../AGENTS.md) — conventions and non-negotiables.
4. [`decisions/README.md`](./decisions/README.md) — the decision log so far.

## Conventions

- Markdown only in this folder. No HTML wrappers.
- Relative links between docs (never absolute URLs to this repo).
- Code references use the inline ``backtick`` form for short names and fenced code blocks with language tags for examples.
- One topic per file; split when a file outgrows ~500 lines.
