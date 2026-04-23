# Echo Codex — Unified Proposal

> A reconciliation and refinement of *Echo Codex project overview.md* and *Eco DND Project Specification.md*, tuned to the real constraints: **free/cheap to operate**, **easy mobile delivery**, **low-friction data entry**, and **optionally system-agnostic**.

---

## 1. Executive Summary

**Echo Codex** is a mobile-first Progressive Web App (PWA) that replaces the paper character sheet and the stack of rulebooks at your side of the table. It stores every piece of information the player needs during a session — character state, spells, items, NPCs, notes — and renders it as fast, one-handed UI.

The two source documents agree on the *shape* of the product but disagree on two important axes:

| Axis              | Echo Codex (doc 1)                                   | Eco DND (doc 2)                        | This proposal                                                                                                                   |
| ----------------- | ---------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Runtime storage   | Individual YAML files on disk                        | SQLite-in-browser (sql.js / wa-sqlite) | **SQLite via wa-sqlite on OPFS**, used as a **hybrid document + relational** store (schemaless JSON fields, SQL indexes + FTS5) |
| Authored format   | YAML                                                 | JSON                                   | **YAML primary** for packs, exports and hand-edits; JSON accepted on input, offered as alternate export                         |
| Data entry        | Community "Parser Mods" scraping TTRPG wikis         | Manual form entry into user templates  | **Manual entry + shareable Template / Data packs** built with internal CLI tooling (§11, phase 4a). No in-app scrapers, no API |
| Hosting           | Central "Parser Repository" cloud API                | None required                          | **None required.** Static hosting only. Packs live at any static URL, including a GitHub repo                                   |
| System-agnostic   | Yes (by design)                                      | Yes (by design)                        | **Yes, via template packs** — ship a D&D 5e pack by default, add other systems as more packs                                    |

This eliminates every component that costs money or legal attention: no scrapers hitting third-party wikis, no server to run, no storage to pay for. The user's data lives on their device; the app is a single static bundle.

---

## 2. Design Principles

1. **Local-first, cloud-optional.** The app works fully offline after first load. The user's data never has to leave their device.
2. **No server, no bill.** Static hosting (Cloudflare Pages / GitHub Pages / Netlify free tier) is the entire backend. $0/month at any realistic scale.
3. **User owns the data.** Runtime storage is a single SQLite file; every piece of data is exportable as human-readable **YAML** (JSON as alternate). A lost phone never loses a campaign.
4. **Templates, not hard-coded rules.** The app is a rendering and tracking engine over user-defined schemas. D&D 5e is a *default content pack*, not a special case in code.
5. **One-handed at the table.** Every frequently-used action is reachable by thumb. Typing is a last resort during a session.
6. **Cheap to add data.** Three tiers of data entry: (a) ship a prebuilt D&D 5e pack, (b) paste-and-split from any text source, (c) form entry via templates. No scraping, no parser maintenance.

---

## 3. Product Scope

### 3.1 MVP (ships first)

- Character sheet renderer driven by templates.
- **Inline editing is first-class.** Tap-to-edit and long-press-to-edit-max on every widget; full form editor behind a long-press menu. Players level up, lose HP, and pick up loot constantly — an MVP that can only read is useless.
- Entity CRUD (create/read/update/delete) for any user-defined entity type (spells, items, NPCs, locations, factions, quests, …).
- Counter system with two classes: **Active** (HP, spell slots, ki — frequent) and **Passive** (XP, gold, level — rare).
- Rest macros (Short Rest / Long Rest / custom) that reset tagged counters.
- Bottom-to-top swipe HUD for Active counters.
- Session notes with markdown + image attachments.
- Full-text search across entities and notes.
- Export / import of the entire DB as **YAML** (or `.db` binary) file.
- A curated **D&D 5e SRD content pack** preloaded on first run (opt-in).
- **Single-character UI**, but the schema, IDs, and navigation model support multiple characters from day one — adding a character picker in V1.1 is a UI change, not a data-model change.

### 3.2 V1.1 — quality of life

- Multi-character picker UI (data already supported in MVP).
- Smart Collections (saved queries: "my prepared spells", "items I own", "NPCs in current city").
- Paste-assist import: paste a stat block or spell text, get a best-effort parse into the current template's fields; user confirms.
- Template editor UI (MVP allows only YAML/JSON editing of templates).
- Optional end-to-end-encrypted sync via the user's own Google Drive / Dropbox (OAuth, no server involvement).

### 3.3 V2 and beyond

- Multi-campaign scoping with cross-campaign entity sharing.
- Collaboration (real-time or async) — explicitly deferred because it breaks the zero-server model. When we get here, look at CRDTs over a cheap managed backend like Cloudflare D1 + Durable Objects.
- Community Content Pack gallery — still just a static YAML/JSON index in a GitHub repo, not a real service.

### 3.4 Explicitly out of scope

- Built-in dice roller physics, VTT features, or initiative trackers for the whole table. Those are different products.
- Scraping third-party TTRPG sites. Legally dubious, operationally expensive, and unnecessary once content packs exist.
- A "Parser Repository" as a hosted service. Replaced by static content packs (§6.3).

---

## 4. System-Agnosticism — how to get it almost for free

Both source documents claim system-agnosticism but underestimate what it costs. The trick is to **push all game-specific knowledge into data, not code.** Concretely:

- The app ships with **zero** hard-coded concepts of "spell", "class", "HP", "AC", "proficiency", "level", or anything else.
- The app **does** know about: entities, templates, fields, counters, collections, rest macros, tags, references.
- A **Content Pack** is a bundle of templates + seed entities + default counters + default rest macros. It is a plain JSON file.
- Default D&D 5e Pack: character template, spell template, item template, 5e SRD spells/items, HP/AC/spell-slot counter definitions, Short/Long rest macros. Shipped with the app; can be ignored/uninstalled.
- Another Pack (e.g. Pathfinder 2e, Call of Cthulhu, Blades in the Dark) is just another JSON file. Users import it from a URL or file.

**Cost of supporting this vs D&D-only:** close to zero, provided we resist the temptation to hard-code anything from day one. The discipline is: *any time you're about to write `if (entity.type === "spell")`, stop and ask whether the template can describe this instead.*

**Recommendation:** build system-agnostic from day one, but only ship the D&D 5e pack at launch. Other packs are a V1.1 concern.

---

## 5. Information Architecture

### 5.1 Core objects

```
Database
├── Templates          // schemas: fields, display hints, counter defs
├── Entities           // instances of templates (characters, spells, items, NPCs, pages, notes)
├── Counters           // live values attached to any entity (usually a character)
├── Macros             // named actions (Short Rest, Long Rest) with tagged resets
├── Collections        // saved queries over Entities
├── Media              // blobs (images) stored in OPFS, referenced by id
└── Settings           // locale, theme, active character, etc.
```

**Character is a normal entity, but anchor-shaped.** It has no special status in the engine. What makes it the "root" is that it owns counters, lists the character's pages, and is the entity the HUD and navigation are scoped to. Everything else (pages, spells, items, NPCs) hangs off it through references. This recursion is what lets the app be data-driven all the way down to the navigation bar.

### 5.2 Template (schema)

A template describes one entity type. It is *data*, editable via a form builder (V1.1) or directly as YAML (MVP).

```yaml
# tpl.spell.5e — minimal D&D 5e spell template
id: tpl.spell.5e
version: 1
name: Spell (5e)
display_name: { en: Spell, uk: Закляття }
tags: [spell]
fields:
  - { key: level,      type: int,      display: bubble }
  - { key: school,     type: enum,     options: [abjuration, conjuration, divination, enchantment, evocation, illusion, necromancy, transmutation] }
  - { key: cast_time,  type: string,   display: subheader }
  - { key: range,      type: string,   display: subheader }
  - { key: components, type: tag_list }
  - { key: duration,   type: string,   display: subheader }
  - { key: body,       type: markdown, display: markdown }
```

**Field optionality rule:** every field is optional by default. Only fields marked `required: true` must be present. The renderer tolerates missing *and* extra fields silently — a partially-filled entity still renders its filled parts. In practice the only required fields will be `id`, `template_id` and `name`.

### 5.3 The Character template (example)

Deliberately minimal — a real D&D 5e pack will carry a richer one, but this is what the engine needs at minimum:

```yaml
# tpl.character.core
id: tpl.character.core
version: 1
name: Character
tags: [character]
fields:
  - { key: name,     type: string,   required: true, display: header }
  - { key: portrait, type: media,                    display: avatar }
  - { key: level,    type: int,                      display: bubble }
  - { key: pages,    type: ref_list, of_tag: character_page, display: page_list }
```

A character instance:

```yaml
# ent.valerius
id: ent.valerius
template_id: tpl.character.core
name: Valerius the Red
fields:
  portrait: media:av-001
  level: 5
  pages: [ent.valerius.combat, ent.valerius.bio, ent.valerius.notes]
```

A **character page** is itself a normal entity. Swiping left/right in the main view moves through this `pages` list. A minimal combat page:

```yaml
# tpl.character_page.combat
id: tpl.character_page.combat
version: 1
name: Combat
tags: [character_page]
fields:
  - { key: hp,    type: counter_ref }
  - { key: ac,    type: int,      display: bubble }
  - { key: stats, type: object, display: stat_grid, fields:
      # Each stat is a pair of plain user-entered fields: score + modifier.
      # No formulas, no derivation — the user writes what they want to see.
      - { key: str_score, type: int,    display: bubble,    display_options: { label: STR, secondary_ref: str_mod } }
      - { key: str_mod,   type: string }
      - { key: dex_score, type: int,    display: bubble,    display_options: { label: DEX, secondary_ref: dex_mod } }
      - { key: dex_mod,   type: string }
      - { key: con_score, type: int,    display: bubble,    display_options: { label: CON, secondary_ref: con_mod } }
      - { key: con_mod,   type: string }
      - { key: int_score, type: int,    display: bubble,    display_options: { label: INT, secondary_ref: int_mod } }
      - { key: int_mod,   type: string }
      - { key: wis_score, type: int,    display: bubble,    display_options: { label: WIS, secondary_ref: wis_mod } }
      - { key: wis_mod,   type: string }
      - { key: cha_score, type: int,    display: bubble,    display_options: { label: CHA, secondary_ref: cha_mod } }
      - { key: cha_mod,   type: string } }
  - { key: spell_slots_1, type: int, display: dots, display_options: { max: 4 } }
  - { key: spells, type: ref_list, of_tag: spell, display: card_list }
```

The "flip" gesture (§7.6) reveals the back side of a page — typically the same page's rarely-used fields — by swapping to a companion template, e.g. `tpl.character_page.bio_back` paired with `tpl.character_page.combat`.

### 5.4 Counter

Counters are a first-class object, not a field, because they have behavior (rest tags, widgets, optional history).

```yaml
id: ctr.valerius.hp
owner_id: ent.valerius
name: HP
kind: active
current: 24
max: 24
display: progress-bar
resets_on: [long_rest]
```

### 5.5 Display types (rendering engine)

A `type` says what the data **is**; `display` says how to **render** it. Multiple displays can consume the same type — this is the key to the flexibility the user asked for. A spell slot count is an `int` rendered as `dots`; a stat is an `int` rendered as a `bubble`; a currency is an `int` rendered as a plain `number`.

**Renderers for scalars:**

| `display`      | Suitable types          | Rendering                                              | Options                                                        |
| -------------- | ----------------------- | ------------------------------------------------------ | -------------------------------------------------------------- |
| `bubble`       | `int`, `float`          | Large circle with primary value + optional secondary line (label above, small secondary value below — e.g. `STR` / `14` / `+2`) | `label`, `secondary_ref` (key of a sibling field to display as secondary), `size` |
| `counter`      | `int`                   | ±N stepper                                             | `step`, `min`, `max`                                           |
| `dots`         | `int`                   | Row of filled/empty dots (spell slots, sanity, stress) | `max`, `filled_glyph`, `empty_glyph`                           |
| `pips`         | `int`                   | Small filled squares (d6 pools, Blades-style)          | `max`                                                          |
| `segments`     | `int`                   | Segmented bar (hunger tracks, clocks)                  | `max`, `segments`                                              |
| `progress-bar` | `int` with `max`        | Horizontal fill bar                                    | colors                                                         |
| `slider`       | `int`, `float` with bounds | Drag to set                                         | `min`, `max`, `step`                                           |
| `number`       | any numeric             | Plain monospace number                                 | format                                                         |
| `header`       | `string`                | Page title                                             | —                                                              |
| `subheader`    | `string`                | Label: value row                                       | —                                                              |
| `label`        | `string`, `enum`        | Badge / chip                                           | color                                                          |
| `avatar`       | `media`                 | Circular image                                         | size                                                           |
| `link`         | `ref`                   | Tappable inline link to another entity (navigates to its detail page) | `style: inline \| chip \| card`, `preview_fields` (keys shown on the card style) |

**Renderers for collections:**

| `display`     | Suitable types            | Rendering                                      | Options           |
| ------------- | ------------------------- | ---------------------------------------------- | ----------------- |
| `tag_list`    | `tag_list`, `enum[]`      | Chips                                          | color map         |
| `card_list`   | `ref_list`                | Vertical scroll of summary cards               | card template     |
| `chip_list`   | `ref_list`                | Compact chips                                  | —                 |
| `table`       | `ref_list`, `object[]`    | Tabular                                        | column list       |
| `page_list`   | `ref_list` of page tags   | Dotted page indicator for swipe navigation     | —                 |
| `stat_grid`   | `object` of numeric       | Grid of bubbles                                | columns           |
| `markdown`    | `markdown`                | Rich text block                                | —                 |

**Choosing a renderer per entity, not just per template.** The template provides a *default* `display`, but an entity can override via `display_overrides`. Example: the default D&D spell-slots field uses `dots`, but a user who prefers numbers can switch it to `counter` on their character without cloning the template. This is how we avoid a combinatorial explosion of near-identical templates.

New display types are added in **code**, not data. The set above is deliberately short. Content packs describe data; renderers are the app's responsibility.

### 5.7 Linking and the Compendium pattern

The engine has exactly two reference types. Every "character has X" relationship in the app is built from them.

- **`ref`** — a single reference to another entity. Rendered as a `link` (inline / chip / card).
- **`ref_list`** — an ordered list of references. Rendered as `card_list`, `chip_list`, `table`, or `page_list`.

**The Compendium + Selection pattern.** Whenever the app needs to model "a character has some of the things a class / book / source offers," use two entities:

1. The **compendium entity** owns the complete catalog. Example: `ent.class.wizard` has `features: ref_list` pointing to every Wizard class feature entity that exists.
2. The **active entity** owns the selection. Example: `ent.valerius` has `class: ref` (→ Wizard) and `selected_features: ref_list` (the features actually picked so far).

Both reference the same underlying feature entities. Nothing is duplicated. The class detail page shows the full catalog; the character page shows the player's curated subset.

**Worked example — class features across levels:**

```yaml
# tpl.class_feature — the schema for a single feature
id: tpl.class_feature
version: 1
name: Class Feature
tags: [class_feature]
fields:
  - { key: name,   type: string, required: true, display: header }
  - { key: level,  type: int,    display: bubble }
  - { key: source, type: ref,    display: link, display_options: { style: chip } }  # back-ref to class
  - { key: body,   type: markdown, display: markdown }
```

```yaml
# ent.feat.wizard.arcane_recovery
id: ent.feat.wizard.arcane_recovery
template_id: tpl.class_feature
name: Arcane Recovery
fields:
  level: 1
  source: ent.class.wizard
  body: "Once per day when you finish a short rest..."
```

```yaml
# ent.class.wizard — compendium entity, owns the full catalog
id: ent.class.wizard
template_id: tpl.class.core
name: Wizard
fields:
  features:
    - ent.feat.wizard.arcane_recovery
    - ent.feat.wizard.spell_mastery
    # ... every wizard feature ever ...
```

```yaml
# ent.valerius — active entity, owns the selection
id: ent.valerius
template_id: tpl.character.core
name: Valerius the Red
fields:
  class: ent.class.wizard
  level: 5
  selected_features:
    - ent.feat.wizard.arcane_recovery
    # ... whatever the player has actually picked ...
```

The character's Features page is a `ref_list` renderer over `selected_features`; each item is a `link` to the feature's full entity page. The class description page renders the catalog the same way, just pointed at `features` instead of `selected_features`.

**Why manual selection, not automatic.** We deliberately avoid any hard-coded "at level 3 wizards get X" logic. That knowledge lives in the rule book, not the app. The user curates their own features by tapping "Add" and picking. Smart Collections (V1.1) can optionally filter the picker by `class == wizard && level <= character.level` for users who want the guided path, but nothing is ever auto-added.

**The pattern scales to everything with the same shape:**

| "Has some of" relationship | Compendium field                     | Selection field              |
| -------------------------- | ------------------------------------ | ---------------------------- |
| Class features             | `ent.class.wizard.features`          | `character.selected_features`|
| Spells known               | pack-level spell list                | `character.known_spells`     |
| Items owned                | pack-level item list                 | `character.inventory`        |
| NPCs met                   | campaign compendium                  | `character.known_npcs`       |
| Languages, proficiencies   | pack-level catalogs                  | `character.proficiencies`    |

**Multi-class** is handled without code changes: `class: ref` becomes `class: ref_list`. `selected_features` is already a flat list of references that can mix features from multiple source classes; the `link` renderer shows each feature's `source` chip so it's visible which class it came from.

**Back-references.** The app maintains a lightweight back-ref index in SQL (`entity_refs(from_id, to_id, field)`). Tapping an entity shows a "Referenced by" panel ("this feature appears in Valerius' selected_features"). This is free from the SQL layer and is the answer to "how do I find everyone who knows Fireball."

---

## 6. Technical Architecture

### 6.1 Delivery: PWA

- **Why PWA, not native:** free to distribute (no App Store / Play Store review, no Apple developer fee), installable to the home screen on iOS and Android, works offline via service worker, one codebase.
- **When to revisit:** if we ever need background sync, push notifications at the table, or deeper device integration. Capacitor can wrap the same PWA into a native shell later with minimal effort.

### 6.2 Stack

| Layer        | Choice                                                        | Rationale                                                               |
| ------------ | ------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Language     | TypeScript                                                    | type safety across template/entity shapes                               |
| UI framework | React + Vite                                                  | doc 2 already picked React; Vite is the fastest PWA-friendly build tool |
| Styling      | Tailwind CSS + Radix UI primitives                            | no design system to maintain, accessible by default                     |
| State        | Zustand (or Jotai)                                            | lightweight, no boilerplate, fine for local-first apps                  |
| Storage      | **wa-sqlite** with OPFS backend                               | real SQL in the browser, persistent, fast queries for Smart Collections |
| Media        | OPFS (Origin Private File System) for blobs, referenced by id | native, no quota negotiation, no IndexedDB churn                        |
| Markdown     | `react-markdown` + `remark-gfm`                               | standard, safe                                                          |
| i18n         | `i18next`                                                     | English + Ukrainian at launch                                           |
| Testing      | Vitest + Playwright                                           | unit + one smoke E2E path on mobile viewport                            |
| Hosting      | Cloudflare Pages (or GitHub Pages)                            | free, global CDN, automatic HTTPS                                       |
| CI           | GitHub Actions                                                | free for public repos                                                   |

### 6.3 Data layer

**Source of truth:** a single SQLite database stored in OPFS. Tables:

```
templates(id, name, schema_json, version)
entities(id, template_id, name, name_lower, tags_json, fields_json, updated_at)
entities_fts  -- FTS5 virtual table for search
counters(id, owner_id, name, kind, current, max, display, resets_on_json)
macros(id, name, resets_json)
collections(id, name, query_json)
media(id, mime, size, path)
settings(key, value_json)
```

**Why SQLite and not a NoSQL store (IndexedDB, Dexie, RxDB, PouchDB, …)?**

The data *is* schemaless per entity — each template defines its own fields — so the instinct to reach for a document store is correct. We honour that instinct at the *schema* level by keeping `fields_json` as an opaque JSON blob. We reach for SQLite only at the *query* level, because:

- Smart Collections and search require real indexes. IndexedDB's query capabilities are famously anemic (compound indexes only on known paths, no joins, no full-text). We'd end up writing our own tiny query engine on top.
- SQLite has **FTS5** built in. Free full-text search across entity names, fields, and notes, with ranking and prefix matching, for zero extra dependencies.
- Rest macros are atomic multi-row writes (one macro touches 6–10 counters). SQL transactions make this a one-liner; an IndexedDB equivalent requires careful object-store choreography.
- SQLite stores JSON natively (`json_extract`, `json_set`, JSON1). So we get the document-store ergonomics *inside* a relational engine — the same pattern as Postgres JSONB, DynamoDB's doc model, or MongoDB with indexed fields.
- A single `.db` file is trivially portable between devices. A NoSQL store has no native export format we'd use.

**So the architecture is:** schemaless documents at the *data* level, SQL at the *index and query* level. Tables have a small fixed set of indexed columns (`id`, `template_id`, `tags`, `name_lower`, `updated_at`) and one `fields_json` column that can hold anything. This is the pattern; the choice of SQLite over, say, DuckDB-wasm is purely about bundle size and maturity in a PWA context.

**Why YAML (not JSON) for authored and exchange format**

JSON is stricter and ubiquitous, but for humans writing packs and inspecting exports:

- YAML drops ~40% of syntax noise (no quotes on keys, no mandatory commas) and makes deep structures readable. Our data *is* deep.
- YAML supports comments. Content-pack authors want these; SRD entries often need `# source: …` footnotes.
- Diffs are cleaner in git — line-per-field, no trailing-comma churn.

YAML's downsides are real but manageable:

- The `js-yaml` bundle adds ~50KB gzipped. Acceptable for a one-time PWA download; we lazy-load it only when importing/exporting.
- YAML's implicit type coercion has sharp edges (`no` → `false`, unquoted dates get parsed). We mitigate with `YAML 1.2` strict mode and schema validation on import.

**Decision:**

- **Runtime:** SQLite (binary, irrelevant to humans).
- **Primary authored & exchange format:** YAML. Content packs, exports, hand-editable templates.
- **Accepted on input:** JSON too, no friction for tools that emit it.
- **Offered on export:** YAML default, JSON alternate.

**Pack format — two kinds, one shape.** Packs share a common envelope but come in two flavours, separated architecturally *and* legally (see §9.2):

- **Template Pack** (`*.tpack.yaml`): contains only `templates`, counter definitions, macros, and page presets. No game-content entities. This is schema and UX — our own creative work. Small (tens of KB). Bundled with the app.
- **Data Pack** (`*.dpack.yaml`): contains `entities` that conform to templates. Spells, classes, items, monsters — things with game text. License inherited from the source. Never bundled; always user-installed on demand.

```yaml
# Envelope common to both pack types
pack:
  type: template          # or "data"
  id: dnd-5e-core
  name: D&D 5e Core Templates
  version: 1.0.0
  license: MIT            # or CC-BY-4.0, OGL-1.0a, etc. for data packs
  author: Echo Codex Team
  source_url: https://github.com/.../dnd-5e-core
  requires: []            # data packs require a template pack; e.g. [dnd-5e-core >=1.0.0]
templates: [ ... ]         # template packs only
entities:  [ ... ]         # data packs only
```

The Pack Manager UI shows the two types with different badges and makes the dependency relationship explicit ("This data pack needs the D&D 5e Core template pack. Install both?"). Importing a pack merges into the user DB and never overwrites user-modified entities without confirmation. Uninstalling a template pack warns if data packs depend on it.

### 6.4 No-backend strategy

- First load: user downloads the static bundle (React app + wa-sqlite wasm). This is the only network call required for normal operation.
- Everything after is local.
- Optional: user connects their Google Drive / Dropbox via OAuth (the browser talks directly to the provider, the app never sees credentials). The app uploads the `.db` file on change. End-to-end encrypted with a passphrase the user chooses — sync providers see ciphertext only.
- Content Packs live as static YAML files on any HTTPS URL. The "gallery" is literally a YAML index file in a GitHub repo that lists packs. Anyone can fork it and PR a new pack. Zero moderation infrastructure needed until we have a problem.

---

## 7. UX / UI

Keep the three-tier reachability model both source documents propose; it's good.

### 7.1 Navigation is data

The main view is not a fixed `<Tabs>` component. It's a renderer over the active character's `pages` ref-list, displayed via the `page_list` renderer. Swiping left/right moves through that list. Pages are entities, so:

- Adding a page to a character is creating an entity and dropping its id into `pages`.
- Reordering pages is reordering that array.
- A content pack can ship a **page layout preset** — e.g. the 5e pack ships `[combat, spells, bio, notes]` as the default page set.
- Different systems ship different page presets (a Call of Cthulhu pack might include a `sanity` page; a Blades-in-the-Dark pack might add `clocks`).

This is the same principle that makes the rendering engine data-driven: navigation is just another level of it. There is no "navigation module" in code — there is a `page_list` renderer and a gesture binder.

### 7.2 Tier 0 — HUD overlay

- Bottom-to-top swipe from anywhere reveals Active counters.
- Each counter renders per its `display` (progress-bar, counter, bubble, dots, pips …).
- Short-tap counter: edit `current` (inline ± or numeric keypad).
- Long-press counter: edit `max`, reset, change display, pin/unpin.
- Dismissable with tap-outside or top-to-bottom swipe.

### 7.3 Tier 1 — horizontal pivot (main)

Default page set — a content pack decides the real list:

1. **Combat / Dashboard** — active character sheet. Stats, current resources, equipped items, prepared spells, actions.
2. **Social / Bio** — backstory, relationships, factions, languages.
3. **Session Notes** — campaign log, newest at top, markdown + image attachments.

Each page can have a **back side** — a companion template with lower-frequency fields (detailed history, personality, appearance). Two-finger flip (§7.6) swaps front ↔ back without losing scroll position.

### 7.4 Tier 2 — management (menu / second-hand)

- Codex / Knowledge Base: browse all entities, filtered by template, tag, or Smart Collection.
- Template editor (V1.1; YAML-only in MVP).
- Content Pack manager (install, update, remove).
- Character switcher (V1.1).
- Settings: language, theme, sync, export / import, danger zone.

### 7.5 Visual language

Per doc 2: modern dark UI, soft gradients, high-radius cards, large tap targets. One light theme for completeness. Respect OS-level dynamic type.

### 7.6 Gesture vocabulary

A lean, one-hand-friendly gesture set. Every gesture has a discoverable fallback (long-press menu), so this can be learned gradually.

| Gesture                          | Context              | Action                                                       |
| -------------------------------- | -------------------- | ------------------------------------------------------------ |
| **Short tap**                    | Counter / int widget | Edit `current` (inline stepper or numeric keypad)            |
| **Long-press**                   | Counter / int widget | Edit `max` (and open the full field editor sheet)            |
| **Long-press**                   | Any widget           | Open field menu: edit, change display, remove, duplicate     |
| **Swipe left / right**           | Main view            | Navigate between pages of the active character               |
| **Two-finger swipe ←→**          | Main page            | Flip the current page (front ↔ back)                         |
| **Swipe up from bottom**         | Anywhere             | Reveal HUD overlay (Tier 0)                                  |
| **Swipe down from top of HUD**   | HUD                  | Dismiss HUD                                                  |
| **Pull down from page top**      | Page                 | Reveal page-level actions (add field, reorder, pin to HUD)   |
| **Tap portrait**                 | Any page             | Open character switcher (V1.1); in MVP opens character edit  |
| **Long-press portrait**          | Any page             | Character menu: duplicate, export, delete                    |
| **Swipe right-edge from right**  | Any page             | Open Tier 2 menu (Codex / Settings)                          |
| **Tap ref (entity link)**        | Anywhere             | Push-navigate into that entity; back gesture returns         |

**Back navigation** uses the platform back gesture (Android) / swipe-from-edge (iOS). Ref-navigation keeps a history stack per session.

**Accessibility:** every gesture has a long-press-to-open-menu equivalent. Users with motor impairments are not locked out.

### 7.7 Input ergonomics

- Every field is inline-editable. Tap to change value, long-press for the full editor.
- Numeric counters use a stepper + tap-to-type.
- Image attach: take photo or pick from gallery, stored in OPFS.
- Voice-to-text is free via the platform keyboard; we don't need to do anything special.
- Paste detection (V1.1): when a user pastes a multi-line block into a markdown field or the quick-add dialog, the app offers to split it into template fields using the current template's hints.

---

## 8. Localization

- i18n keys for all UI strings.
- Templates carry a `display_name` object keyed by locale (per doc 1, §2.D) so `str` renders as `Сила` in Ukrainian.
- Entity content (spell descriptions etc.) is not auto-translated. If a user wants a Ukrainian Fireball, they translate it once and save it as that entity's body. A future "alternate language body" field can live on entities if demand is real.
- Side-by-side translation helper (doc 1, §2.D) is a nice-to-have but not MVP.

---

## 9. Licensing and monetization

### 9.1 Code — open source, permissive

- **License: MIT.** Apache 2.0 is the only alternative worth considering; its patent grant is irrelevant for a TTRPG tool and it's more paperwork. MIT is shorter, better understood, and widely adopted in the JS ecosystem.
- **Why permissive, not GPL/AGPL.** We don't run a service, so there's no SaaS position to defend. Permissive licenses maximise adoption, forks, and trust. Anyone can fork and host their own build; that's a feature, not a problem.
- **Governance.** Start as a small-team / BDFL project. Adopt a contributor ladder if the community grows.

### 9.2 Template Pack vs Data Pack — the architectural and legal separation

This is the same split introduced in §6.3, restated here because it's a load-bearing legal principle: **we never ship or monetise game content we don't own.**

|                          | Template Pack (`*.tpack.yaml`)                          | Data Pack (`*.dpack.yaml`)                                         |
| ------------------------ | ------------------------------------------------------- | ------------------------------------------------------------------ |
| Contents                 | Templates, counters, macros, page presets, empty forms  | Concrete entities (spells, items, classes, monsters) with game text|
| Represents               | Our creative work (schema + UX)                         | Third-party content (SRD, OGL, homebrew, etc.)                     |
| IP                       | Ours                                                    | Source's                                                           |
| Bundled with app?        | **Yes** — small (tens of KB)                            | **No** — never                                                     |
| How user gets it         | First-run, one tap                                      | User action (install from URL / file)                              |
| Our default license      | MIT or CC-BY-4.0                                        | Inherited from source (OGL-1.0a, CC-BY-4.0, custom, …)             |
| Can we monetise it?      | Yes, in principle — it's our work                       | **Only if we own the data.** We don't, for any major TTRPG system  |

**In practice:** the app ships with the D&D 5e **template** pack (our schema) and no data. A fresh install is an "app builder with a pre-made 5e skeleton." Separately, on our GitHub, we publish a D&D 5e **SRD data pack** under CC-BY-4.0 (tracking Wizards' 2023 CC release of SRD 5.1), one-click install from URL. Users can also author and share their own data packs under any license they choose.

This separation is the entire legal posture: every TTRPG system becomes compatible by shipping schemas only, and the moment we touch game content we inherit its license without modification.

### 9.3 Monetization strategy

**Principles:**

1. The app is always free and fully featured. No premium tier, no feature lock, ever.
2. All monetization is voluntary. Users who find the app useful can support it; users who can't or don't want to are not penalized in any way.
3. We never monetize data we don't own.
4. Open source is also insurance — if the maintainer stops, the community can continue, and users' hundreds of hours of campaign data stay accessible.

**Tier 1 — Donations (from launch).**

- **GitHub Sponsors** (0% fees, integrates with the repo, works for monthly and one-time).
- **Ko-fi** or **Buy Me a Coffee** for one-off tips from people who don't want to link GitHub.
- A single "Support the project" link on the About screen. **No pop-ups, no timers, no nag banners.** The link is present; it is never pushed.
- Optional, opt-in supporter list on the About screen (names pulled from GitHub Sponsors tiers).

**Tier 2 — Recurring support via Patreon (if traction warrants).**

- Small monthly tiers where the value is *community and influence*, never software features:
  - Name in credits.
  - Early access to development builds.
  - A vote on roadmap polls.
  - Access to a community Discord for design discussion.
- Typical yield for a well-loved indie TTRPG tool: low hundreds to low thousands of dollars per month. Plan for the lower end and treat anything above as bonus.

**Tier 3 — Original template packs we design ourselves (optional).**

- Template packs for indie TTRPGs where we've done the design work. Because the pack contains our schema and zero third-party data, we can legally sell it.
- Priced as impulse purchases ($2–$5, one-time). Purchased through Ko-fi, Gumroad, or itch.io; the app just imports the downloaded file.
- Only viable for systems we genuinely want to design for. Not a core strategy — a side revenue lane.

**Tier 4 — Commissions and services.**

- A TTRPG publisher commissions us to produce an official template pack for their game (B2B).
- Bespoke feature work for TTRPG conventions, studios, or streamers.
- Opportunistic; not planned-for revenue.

**Explicitly ruled out:**

- Ads in the app.
- Paywalling core features.
- Selling SRD, OGL, or other third-party content as if it were ours.
- Running a paid cloud service (violates the zero-server principle).

**Realistic expectations:** this will not pay a full-time salary. It is a passion project that can be self-sustaining if it finds its audience. If donations and Patreon together cover domain + an occasional coffee, call it a win.

---

## 10. Cost model

| Item                             | Cost                                              |
| -------------------------------- | ------------------------------------------------- |
| Hosting (Cloudflare Pages)       | $0                                                |
| Domain (optional)                | ~$10/year (or free on a `*.pages.dev` subdomain)  |
| CI (GitHub Actions, public repo) | $0                                                |
| App Store / Play Store           | $0 (PWA, no store listing needed)                 |
| Template + data pack hosting     | $0 (GitHub repo)                                  |
| Backend / database               | $0 (none)                                         |
| Sync                             | $0 (user's own Drive / Dropbox quota)             |
| Donations platform fees          | 0–5% of donations (GitHub Sponsors 0%, Ko-fi ~5%) |

**Total recurring cost: $0–$10/year.** If the app gets big enough that Cloudflare Pages' free tier limits hurt (500 builds/month, unlimited bandwidth), that's a happy problem.

---

## 11. Roadmap

Rewrites both source roadmaps to match this architecture.

### Phase 0 — Scaffolding (≈1 week)

- Vite + React + TS + Tailwind + Radix.
- PWA manifest, service worker, offline shell.
- wa-sqlite with OPFS, schema migrations framework.
- Deploy CI to Cloudflare Pages.

### Phase 1 — Core data model & renderer (≈2–3 weeks)

- Templates & entities CRUD (read + **edit**, both from MVP day one).
- Display-type renderers: `bubble`, `counter`, `dots`, `pips`, `segments`, `progress-bar`, `header`, `subheader`, `label`, `avatar`, `tag_list`, `markdown`, `ref_list`, `card_list`, `page_list`, `stat_grid`.
- Character template + character-page sub-entities wired through the `page_list` renderer.
- Navigation driven by the active character's `pages` (no hard-coded tabs).
- Inline editing: short-tap / long-press / sheet editor for every field.
- YAML (primary) + JSON (alternate) export/import.

### Phase 2 — Counters, macros, HUD (≈2 weeks)

- Counter engine (active / passive).
- Rest macros with tagged resets.
- Tier 0 swipe HUD overlay.
- Full gesture set (§7.6) — swipe-between-pages, two-finger flip, long-press-to-edit-max.

### Phase 3 — Session notes & search (≈1–2 weeks)

- Markdown notes with image attachments (OPFS).
- FTS5 full-text search across entities and notes.
- Default page preset: Combat / Bio / Notes.

### Phase 4a — Pack-builder tooling (≈1–2 weeks, internal only)

These tools are **never shipped to users.** They are developer-side CLI utilities used to produce prebuilt packs. They live in a separate `packtools/` folder of the repo (or a separate private repo), run offline, and emit `*.tpack.yaml` or `*.dpack.yaml` files.

- **`codex-pack new <id> --type {template|data}`** — scaffold a new pack with the appropriate skeleton.
- **`codex-pack scrape <source>`** — source-specific scrapers (D&D 5e SRD, Open5e, etc.). Each scraper is a small Python or TypeScript module with a uniform interface: input = source URL / dump / file, output = normalized entity chunks. Scrapers only run against sources with a clear open license (SRD under OGL / CC-BY, Open5e under CC-BY, etc.).
- **`codex-pack validate <file>`** — schema-validates a template or data pack; for data packs, checks all `template_id` references resolve against a specified template pack, catches orphan refs, flags display incompatibilities.
- **`codex-pack merge <a> <b>`** — combine partial packs (one per source) into a single shippable pack.
- **`codex-pack build <file>`** — minify, version-stamp, and emit the final pack file consumable by the app.

**What the existing `spells_scraper.py` becomes:** a starting point for the SRD-spells scraper module in this toolchain.

Acceptance criterion: starting from zero, a developer can run `codex-pack scrape open5e --out spells.yaml && codex-pack build ... > dnd-5e-srd.dpack.yaml` and the app can load that data pack against the shipped `dnd-5e-core.tpack.yaml`.

### Phase 4b — D&D 5e Core Template Pack, bundled (≈1 week)

- Produce `dnd-5e-core.tpack.yaml` — templates only. Character, Character Pages (Combat / Bio / Notes), Spell, Item, Monster, Class, Class Feature, Background. Plus the default page preset and the standard counter set (HP, spell slots 1–9, hit dice).
- Bundled with the app binary. First-run flow: "Set up for D&D 5e? [Yes / Pick another system / Empty app]."
- Licensed MIT or CC-BY-4.0 (our work, no third-party content).

### Phase 4c — D&D 5e SRD Data Pack, published separately (≈1–2 weeks)

- Using Phase 4a tooling, produce `dnd-5e-srd.dpack.yaml` — SRD spells, items, monsters, class features, backgrounds.
- Published as a static file on a GitHub repo, **not bundled** with the app.
- Licensed CC-BY-4.0, tracking the 2023 CC release of SRD 5.1.
- App's first-run flow offers: "Install the D&D 5e SRD data pack? [Yes / No / Later]" — a one-tap fetch from the published URL.

### Phase 5 — Smart Collections & template editor (≈2 weeks)

- Saved queries UI over the SQL engine.
- Template editor UI (form-builder) replacing raw YAML editing.
- Multi-character picker UI.

### Phase 6 — Sync & sharing (≈2–3 weeks)

- Google Drive / Dropbox sync with client-side AES-GCM encryption.
- Pack install-from-URL (both template and data packs).
- Shareable entity export (single entity as a YAML file → QR code → import).

### Phase 7 and later

- Additional system template packs (Pathfinder 2e, Call of Cthulhu, Blades in the Dark, Mausritter, …) — produced with the Phase 4a tooling.
- Corresponding data packs where the content is openly licensed.
- Paste-assist importer.
- Native wrap via Capacitor, if push notifications or stricter device integration are ever needed.

---

## 12. Decisions made and remaining open questions

### Decided

1. **Multi-character.** DB schema, IDs, and navigation support multiple characters from day one. UI in MVP exposes one character at a time (single-character UI). V1.1 adds the picker. Multi-campaign scoping is a V2 concern.
2. **Template versioning.** Every template carries `version: <int>`; every entity carries the template version it was written against. Renderer tolerates missing and extra fields (see §5.2 optionality rule). A migration record per version bump is authored by the pack owner and offered to the user on pack update. Bake the version field in from day one.
3. **Field optionality.** All fields are optional by default. Only `id`, `template_id` and `name` on entities are required; everything else renders if present, is silently skipped if absent. This is how we keep the app usable with half-filled characters.
4. **Runtime vs authored format.** SQLite at runtime (hybrid doc+relational), YAML as the primary authored/exchange format, JSON accepted on input and offered as alternate export. See §6.3.
5. **Paste-assist.** Deferred to V1.1. The rules-agnostic text-splitter plus per-template hint regexes approach is sound but not needed for MVP.
6. **Navigation.** Data-driven. The main view is a `page_list` renderer over the active character's `pages` ref-list. No hard-coded tabs. See §7.1.
7. **Gestures.** Locked in per §7.6. Short-tap edits current, long-press edits max, horizontal swipe navigates pages, two-finger swipe flips a page front/back, bottom-up swipe raises the HUD.
8. **Licensing posture.** Code under **MIT**. Template packs under MIT or CC-BY-4.0 (our work). Data packs carry their source license (e.g. CC-BY-4.0 for the 5e SRD). App never ships bundled game content. See §9.
9. **Monetization.** Donations (GitHub Sponsors + Ko-fi) from launch; optional Patreon tiers later; possibly paid original template packs; commissions opportunistic. No ads, no premium tier, no selling third-party data. See §9.3.
10. **Pack architecture.** Two pack types — template packs (`*.tpack.yaml`, ours, bundleable) and data packs (`*.dpack.yaml`, source-licensed, user-installed). Data packs declare a `requires: [template-pack-id]` dependency. See §6.3 and §9.2.
11. **Entity linking.** Two reference types (`ref`, `ref_list`) plus a `link` renderer. The **Compendium + Selection** pattern models every "character has some of the things a book offers" relationship — same pattern for features, spells known, inventory, etc. Back-references indexed in SQL. Manual selection only; no hardcoded class/level automation. See §5.7.

### Open — separate discussions

1. **Exact license per template pack and per data pack.** The posture is clear (MIT/CC-BY for ours, inherit for third-party), but the per-pack license declaration needs to be walked through before Phase 4b ships — especially OGL 1.0a vs CC-BY-4.0 for the 5e SRD, and attribution requirements on Open5e. Not app-architecture-critical; resolve before first public release.
2. **Derived / computed fields — deferred.** Early versions will **not** evaluate formulas at all. Where two values exist (e.g. D&D ability score + modifier), the template just declares them as a pair of plain user-entered fields (see the `stats` block in §5.3). The user is responsible for keeping them consistent; the app is pure storage and display. This removes a whole category of complexity: no expression parser, no sandbox, no untrusted-code surface from content packs, no arithmetic errors to debug, and no performance cost on renders. Revisit only if users consistently ask for auto-computed derivations — at that point evaluate `expr-eval` (tiny, whitelist-able) vs a ~200-line custom parser. Not on the MVP path.
3. **Template migration ergonomics.** Decided *that* we version; still open *how* migrations are authored. Options: imperative mini-scripts shipped with the pack, declarative field-rename/field-add rules, or just "additive-only" versioning (forbid breaking changes). Lean toward additive-only for community packs, with an escape hatch for our own.
4. **"Add feature" picker filter logic (V1.1).** When Smart Collections arrive, should the character's "Add feature" picker default-filter to features matching the character's class(es) and level, or show everything unfiltered? Defer the UX choice; the underlying Smart Collection mechanism supports both.

---

## 13. What changed vs. the source documents

**Kept from Echo Codex (doc 1):**

- Three-tier reachability model.
- Display-type rendering engine and the specific display types.
- System-agnostic template approach.
- Recursive entity linking (now via IDs instead of filenames).
- Localized `display_name` on templates.

**Kept from Eco DND (doc 2):**

- PWA + React + TypeScript stack.
- SQLite as runtime storage (but with wa-sqlite + OPFS, not sql.js).
- Typed counter system (active vs passive) and rest macros.
- Form-builder for templates.
- Modern dark UI direction.

**Dropped:**

- **Parser Repository as a hosted cloud API** (doc 1). Replaced by static Content Packs on any URL. Removes the single component that had real recurring cost and legal risk.
- **Per-entity YAML files on the filesystem** (doc 1). Replaced by SQLite-in-OPFS (document-oriented via JSON columns) with YAML/JSON export. Keeps portability, gains performance and search.
- **Parser Mods that scrape third-party wikis at runtime** (doc 1). Replaced by developer-side Pack Builder CLI tooling (§11 phase 4a) that produces prebuilt packs offline.

**Added:**

- Explicit cost model ($0 recurring) and monetization strategy (voluntary donations, optional Patreon, optional original template pack sales, no ads, no premium tier, no selling third-party data).
- **Open-source posture** — MIT-licensed code, permissive licensing for our template packs.
- **Template Pack vs Data Pack split** as both an architectural file format distinction and a load-bearing legal boundary: app ships with template packs (ours), never bundles game content. Data packs are user-installed, source-licensed.
- **Developer-side Pack Builder CLI** as a distinct phase (never shipped to users) — this is where scrapers live now.
- **Compendium + Selection pattern** for entity linking: `ref` / `ref_list` types plus a `link` renderer. Solves class-features-by-level, spells-known, inventory, and multi-classing without any game-specific code.
- **Back-reference index** in SQL so "who references this entity?" is free.
- **Data-driven navigation**: the main view renders an entity's `pages` list; no hard-coded tabs.
- **Full gesture vocabulary** (short-tap edits current, long-press edits max, two-finger flip, etc.), with discoverable long-press fallbacks for accessibility.
- **Expanded renderer set** that lets the same data type (e.g. `int`) display as bubble, counter, dots, pips, segments, bar, or plain number — with per-entity overrides.
- **Character template** as a concrete example of the anchor entity, with sub-page entities.
- **Field optionality rule**: everything optional by default except `id`, `template_id`, `name`.
- **YAML as primary authored format** (JSON accepted), with rationale in §6.3.
- Optional end-to-end-encrypted sync via user-owned Drive/Dropbox, not a service we run.
- Template versioning from day one.
- A sharper line between MVP, V1.1, and V2, so the first release is actually shippable.
