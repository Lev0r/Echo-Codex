# Echo Codex

> A free, open-source, mobile-first Progressive Web App that replaces the paper character sheet and the stack of rulebooks at your side of the table. System-agnostic by design; ships with D&D 5e templates out of the box.

**Status:** pre-alpha. The project is currently in the design phase — no app code yet. The full design is in [`docs/spec.md`](./docs/spec.md).

## What it is

Echo Codex is a template-driven renderer and tracker for tabletop RPG data. Instead of hard-coding any one game system, it defines:

- **Templates** that describe entity shapes (characters, spells, items, class features, …).
- **Data-driven UI** that renders those entities as one-handed, gesture-friendly mobile pages.
- **Two kinds of packs** — **Template Packs** (schemas, ours, bundled) and **Data Packs** (game content, third-party, opt-in, never bundled). This separation is both architectural and legal.

The app runs as a PWA (install from the browser, works offline), stores everything locally in SQLite via OPFS, and never talks to a backend unless the user opts into sync via their own Google Drive or Dropbox.

## Why

- **Free forever.** Permissive MIT license on the code, no premium tier, no ads.
- **Cheap to operate.** Static hosting only; $0–$10/year recurring cost.
- **Your data is yours.** Local-first, fully exportable as human-readable YAML.
- **Any system.** D&D 5e at launch; other systems arrive as template packs, not forks.

## Documentation

- [Unified design spec](./docs/spec.md) — the single source of truth for architecture, UX and roadmap.
- [AI agent guide](./AGENTS.md) — conventions for human and AI contributors.
- [Architecture Decision Records](./docs/decisions/) — the log of significant choices.
- [Developer guides](./docs/guides/) — setup, pack authoring, release process (growing).

## Support

This is a passion project. If it turns out to be useful to you, support is always voluntary — never required and never pushed. Donation links will appear on the About screen once the app ships.

## License

Code: [MIT](./LICENSE). Our template packs: MIT or CC-BY-4.0 (per pack metadata). Data packs inherit their source license.
