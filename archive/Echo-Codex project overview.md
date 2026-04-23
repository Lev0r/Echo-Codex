# Echo Codex: Project Specification

## 1. Project Philosophy
**Echo Codex** is built on three core pillars:
1.  **Rules-Agnosticism:** A high-quality rendering engine for structured TTRPG data with zero hard-coded game logic.
2.  **Privacy & Ownership:** Data belongs to the user, stored as human-readable YAML files.
3.  **Modular Extensibility:** Offload the complexity of data entry to a community-driven "Mod" ecosystem. Instead of manual entry, the app consumes data from external sources via specialized parsers.

---

## 2. Core Features & Workflows

### A. The "Mod-Driven" Knowledge Base
Echo Codex treats information as **Structured Entities** (YAML). To populate the Codex, users use "Mods" to import data from their favorite TTRPG websites and sources.

**The Import Workflow:**
1.  **Source Enablement:** User opens the **Parser Repository** (a cloud-hosted library of community-tested mods).
2.  **Add Mod:** The user finds a mod for a specific source (e.g., *"5e Spell Wiki Parser"*) and clicks **(+) Add**.
3.  **URL/Data Input:** The user provides a URL or raw text from that specific source.
4.  **Remote/Modular Parsing:** The app uses the mod's logic (potentially a cloud-based scraper/API) to transform the source data into an Echo Codex YAML file.
5.  **Finalization:** The structured YAML is saved locally, ready for rendering and character management.

### B. Dynamic Layout Engine (Widget Mapper)
The core app is a premium renderer for YAML fields. It uses **Display Types** to decide how to show data:
- **Header:** Page titles.
- **Subheader:** Labeled descriptors (e.g., "Cast Time").
- **Stat Bubble / HP Bar / XP Bar:** High-visibility circular or linear widgets for tracking.
- **Inventory / Spellbook:** Recursive lists that "unroll" linked entities.
- **MarkdownBlock:** For lore and long descriptions.

### C. Smart Collections (Filtered Views)
Virtual lists created by querying local YAML files.
- **Spellbook:** A view of all entities where `template == "Spell"` and `is_prepared == true`.
- **Inventory:** Entities where `template == "Item"` and `owner == "CharacterID"`.

### D. Localization & Translation Workflow
Echo Codex is designed for multi-language support (e.g., Ukrainian) without expensive server-side processing:
- **Localized Mods:** The Parser Repository can host mods specifically designed for localized sources (e.g., a parser for a Ukrainian TTRPG wiki).
- **Side-by-Side Helper:** For English-only sources, the app provides a translation UI. User imports English data $\rightarrow$ Uses browser-native "Translate to Ukrainian" $\rightarrow$ App captures the translated strings and saves them to the local YAML.
- **Display Keys:** Templates support `display_name` keys so a system-agnostic key like `str` can be rendered as `Сила` in the UI.

---

## 3. Technical Architecture

### A. Data Persistence (Local-First)
- **Source of Truth:** Individual **YAML files**.
- **In-Memory Indexing:** Metadata is loaded into a lightweight in-memory index for fast searching and collection generation.

### B. Parser Repository (Cloud API)
A central API that hosts and serves "Parser Mods."
- **Mod Logic:** Contains the regex, scraper selectors, or transformation logic required to convert a specific website's HTML/text into the Echo Codex YAML schema.
- **API Access:** The application fetches the required mod logic on-demand to perform imports.

### C. Widget Behavior (Display Types)
The Layout Engine maps YAML keys to specific React components based on the `display` attribute:
- **`bubble`**: A circular widget for primary stats (e.g., Strength). Displays the raw number + a calculated modifier if configured.
- **`progress-bar`**: A horizontal bar for tracking resources (e.g., HP). Requires `current` and `max` keys in the YAML.
- **`counter`**: A compact widget with +/- buttons for high-frequency tracking (e.g., Spell Slots).
- **`markdown`**: A full-width block that renders rich text, bolding, and lists.

### D. UI/UX Reachability Hierarchy
Designed for one-handed table use:
- **Tier 0 (Overlay):** Resource HUD (Active Counters). Bottom-to-top swipe.
- **Tier 1 (Main):** Horizontal Swipe-to-Pivot.
    - *Combat:* The active character "Dashboard."
    - *Social/Bio:* Character background.
    - *Session Notes:* Campaign log.
- **Tier 2 (Management):** Codex, Parser Repository (Mods), Template Settings.

---

## 4. Deep Architecture Details

### A. Recursive Entity Linking
Entities reference each other by filename.
- **Character (`valerius.yaml`)** points to an `inventory` list.
- **Items** in the inventory are separate YAML files.
- The UI "unrolls" these links to create a seamless experience.

### B. Media Management
- **Local Assets Folder:** Images are stored in an `/assets` folder.
- **Linking:** YAML fields store relative paths: `avatar: "./assets/wizard.png"`.

---

## 5. Development Roadmap

- **Phase 1: Pure YAML Foundation.** Implement YAML parsing, local file system access, and the base rendering engine.
- **Phase 2: Mod Architecture & Parser API.** Build the system to fetch and execute external parser logic.
- **Phase 3: The Layout Engine.** Implement the "Display Type" system (Bubbles, Bars, Headers).
- **Phase 4: Search & Collections.** Implement in-memory indexing and filtered views.
- **Phase 5: Ecosystem & Cloud Sync.** Launch the Parser Repository and Google Drive/Dropbox integration.

---

## 6. Appendix: Concrete Examples

### Example: A Character Entity (`valerius.yaml`)
```yaml
template: "character-template"
name: "Valerius the Red"
stats:
  str: 8
  dex: 14
  int: 18
counters:
  hp: { current: 24, max: 24, display: "progress-bar" }
inventory:
  - "staff-of-power.yaml"
  - "bag-of-holding.yaml"
spellbook:
  - "fireball.yaml"
```
