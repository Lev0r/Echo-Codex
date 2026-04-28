# AGENTS.md — Guide for AI and human contributors

This file is the **canonical source of truth for agent behaviour** in this repository. It is read by Cursor, Claude Code, Copilot, and any other AI tool that follows the `AGENTS.md` convention, and it is equally useful to a new human contributor.

Keep it short. When it starts to bloat, extract topics into `docs/` and link to them.

---

## 1. What this project is — in one paragraph

**Echo Codex** is a free, open-source, mobile-first PWA that replaces the paper character sheet and the stack of rulebooks at a tabletop-RPG table. It is a **data-driven rendering and tracking engine** over user-defined templates — system-agnostic by design, D&D 5e at launch. The full design is in [`docs/spec.md`](./docs/spec.md) and is the single source of truth. **Read it before making non-trivial decisions.**

## 2. Non-negotiables

These follow from the spec and must not be violated without an ADR documenting the change (§6).

1. **No backend, ever (at MVP).** The app is a single static PWA bundle; sync is via the user's own Drive/Dropbox only. If you find yourself reaching for a server, stop and open a discussion.
2. **Local-first.** User data lives in SQLite on OPFS on the device. Nothing leaves the device unless the user explicitly opts in.
3. **Templates, not hard-coded rules.** No `if (entity.type === "spell")` in app code. Game-specific knowledge lives in template and data packs, never in `app/src`.
4. **Template Pack vs Data Pack separation is load-bearing and legal.** Ship only our own schemas bundled. Game content (spells, classes, items) goes in data packs, never bundled, user-installed. See spec §6.3 and §9.2.
5. **MIT-licensed code.** No dependencies with GPL/AGPL/commons-clause licences without prior discussion.
6. **YAML is the primary authored format.** JSON is accepted on input and offered as alternate export.
7. **Editing is first-class from MVP.** Never ship a read-only surface for user data.
8. **Every field is optional** except `id`, `template_id`, `name`. The renderer tolerates missing and extra fields silently.
9. **No content packs contain executable code.** No formulas, no scripts, no hooks. If this rule seems limiting, re-read spec §5.5 / §11.

## 3. Repository map

| Path                     | What lives here                                                          | AI-editable? |
| ------------------------ | ------------------------------------------------------------------------ | ------------ |
| `docs/INDEX.md`          | Hand-curated map of every doc, organised by intent. The navigation surface. | Yes — keep in sync when docs are added/renamed |
| `docs/spec.md`           | The unified design spec. Single source of truth.                         | Yes, carefully — see §5 |
| `docs/decisions/`        | Architecture Decision Records (ADRs), numbered, immutable once accepted. | Add only; never edit accepted ADRs |
| `docs/architecture/`     | Living technical documentation.                                          | Yes          |
| `docs/guides/`           | How-tos (setup, pack authoring, release).                                | Yes          |
| `docs/notes/`            | Agent-writable memory: open questions, exploration logs, session summaries. **Not load-bearing.** | Yes — see `docs/notes/README.md` |
| `app/`                   | The PWA source. *Scaffolded in Phase 0, not yet present.*                | Yes          |
| `packtools/`             | Developer CLI for building packs. *Scaffolded in Phase 4a.*              | Yes          |
| `packs/`                 | Source for our own-authored template packs. *Created in Phase 4b.*       | Yes          |
| `archive/`               | Historical, superseded documents. **Do not reference or reintroduce ideas from here.** | **No — out of scope.** |
| `.cursor/rules/`         | Tool-specific rule stubs. Keep thin; substance goes in this file.        | Yes          |
| `CLAUDE.md`              | Thin shim redirecting Claude Code to this file. Three lines.             | Yes          |
| `.github/copilot-instructions.md` | Thin shim redirecting GitHub Copilot to this file. Three lines.  | Yes          |
| `AGENTS.md`              | This file.                                                               | Yes, sparingly |
| `README.md`              | Public-facing. Keep short.                                               | Yes          |

`.cursorignore` excludes `archive/` and build artefacts from AI indexing. Don't add content to `archive/`; it's a write-once attic.

## 4. Documentation discipline

We let AI maintain docs, which only works if the discipline is clear.

- **The spec is the anchor.** Changes to design direction update `docs/spec.md`. When you change it, add a short entry to `docs/decisions/` pointing to the section(s) that changed and why.
- **ADRs are immutable.** Once an ADR's status is `Accepted`, do not rewrite it. If we change our mind, add a new ADR that `Supersedes: ADR-XXXX`.
- **Architecture docs are living.** Under `docs/architecture/`, feel free to rewrite freely. Keep each file focused on one topic.
- **Guides are task-oriented.** "How do I…?" — setup, pack authoring, releasing. Not background; background goes in `architecture/`.
- **Notes are observations, not commitments.** Use `docs/notes/` for open questions, exploration logs, and session summaries. Notes are explicitly **not load-bearing** — when a note disagrees with the spec or an ADR, the latter wins. Promote a note rather than expand it: open question → ADR; exploration → architecture doc; spec changes only via an ADR. See `docs/notes/README.md` for the policy.
- **Don't duplicate the spec.** If a topic is covered there, link to the section. Documentation rot kills projects; single-source everything.
- **Keep `docs/INDEX.md` in sync.** Add an entry whenever a doc is created, renamed, or deprecated. The one-liner there should match the doc's frontmatter `summary:`.
- **Don't create new top-level docs folders** without prior discussion — propose in a PR.

## 5. When editing the spec

`docs/spec.md` is long. Editing protocol:

1. Read the relevant section(s) fully before changing them.
2. Prefer targeted edits over rewrites.
3. If a change crosses section boundaries, also update §13 *(What changed vs. the source documents)*.
4. If the change is meaningful (affects architecture, scope, monetization, licensing, or user-visible behaviour), also add an ADR in `docs/decisions/`.
5. Never delete the numbered section headings without renumbering references throughout the file.

## 6. Architecture Decision Records

Template and instructions live in [`docs/decisions/README.md`](./docs/decisions/README.md). Write one when:

- Introducing a new third-party dependency of any size.
- Changing a non-negotiable from §2.
- Deciding between materially different technical approaches.
- Adding a new pack type, file format, or storage layer.

Don't write one for routine implementation; that's over-process.

## 7. Coding conventions

The codebase doesn't exist yet. When Phase 0 lands, language- and framework-specific rules will live in `.cursor/rules/typescript.mdc`, `.cursor/rules/react.mdc`, etc., and be linked from here. Until then, no code conventions apply.

The spec §6.2 names the intended stack (React + Vite + TS, Tailwind + Radix, wa-sqlite on OPFS, Zustand, react-markdown, i18next, Vitest + Playwright). Don't substitute without an ADR.

## 8. Commits, branches, PRs

- **Conventional Commits** from commit #1: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, `test:`, optionally scoped (`feat(packs): …`).
- **One logical change per commit.** Don't bundle unrelated changes.
- **Branches:** `main` is always shippable. Work branches are `type/short-description` (e.g. `feat/pack-loader`, `docs/spec-character-template`).
- **PRs:** small, focused, with a short rationale. Squash-merge into `main`.
- **Never force-push `main`.**
- **Never commit secrets.** `.gitignore` covers the obvious cases; still read the diff before staging.

## 9. When in doubt

- **Don't invent scope.** The spec and the roadmap are the plan of record. If a request conflicts with them, ask.
- **Prefer the smallest change that answers the prompt.** Then stop.
- **Keep the archive closed.** If old docs contradict the spec, the spec wins.
- **Ask one clarifying question** rather than guess at ambiguous requirements; don't ask five.

## 10. Memory layers

The project keeps four documentation layers, each with a single, well-defined purpose. Pick the right one before writing:

- **Rules** live in this file (`AGENTS.md`). `.cursor/rules/*.mdc`, `CLAUDE.md`, and `.github/copilot-instructions.md` are thin shims that point back here. Do not duplicate rules across them.
- **Design** lives in `docs/spec.md`. The single source of truth for architecture, UX, roadmap, licensing, monetization. Section-numbered; edits follow §5.
- **Decisions** live in `docs/decisions/` as ADRs. Immutable once accepted; supersede with a new ADR.
- **Architecture** lives in `docs/architecture/`. Living technical docs that evolve with the code.
- **Exploration** lives in `docs/notes/`. Agent-writable, **not load-bearing**, promotable to ADR / architecture / spec.

Navigation across all layers is in [`docs/INDEX.md`](./docs/INDEX.md). The compressed map injected into every Cursor session is in [`.cursor/rules/docs-map.mdc`](./.cursor/rules/docs-map.mdc).

When unsure where something belongs: if it is a *commitment*, it goes in `spec.md` (with an ADR for the decision). If it is an *observation*, it goes in `notes/`. If it is *implementation detail*, it goes in `architecture/`. If it is a *how-to*, it goes in `guides/`.
