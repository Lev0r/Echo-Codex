# Architecture

Living technical documentation. Unlike the spec (which is the canonical high-level design) and ADRs (which are immutable decisions), files here evolve with the code.

**Nothing lives here yet.** Documents will be added as the project matures. Anticipated topics:

- `data-model.md` — concrete SQLite schema, JSON field conventions, migrations.
- `rendering-engine.md` — how templates become React components, renderer registry, display-override resolution.
- `navigation.md` — data-driven navigation, gesture binding, back-stack.
- `pack-loading.md` — how template / data packs are parsed, validated, merged into the user DB.
- `sync-protocol.md` — Drive/Dropbox sync design, AES-GCM envelope, conflict resolution.
- `performance-budget.md` — target metrics (bundle size, TTI, search latency) and how we track them.

When writing a new architecture doc:

1. Link it from this index.
2. Keep it focused on a single subsystem.
3. Link back to the spec section it implements, not the other way round.
4. If the doc records a decision rather than an implementation, it's an ADR — put it in `../decisions/` instead.
