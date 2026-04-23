Eco DND: Project Specification

1. Project Overview

Eco DND is a rules-agnostic Progressive Web Application (PWA) designed as a digital replacement for pen-and-paper Tabletop RPG character sheets. It utilizes a user-driven "Knowledge Base" approach, eliminating predefined rulesets in favor of absolute customization and manual data entry via dynamic templates.

2. Core Features

System-Agnostic Character Management: Digital sheet allowing manual overrides for all stats, inventory, and currency. Fields are user-generated based on custom configuration.

User-Defined Knowledge Base (Wiki System):

Replaces pre-populated reference libraries.

Users build compendiums (e.g., Spells, Pantheons, Status Effects) by creating structured pages.

Dynamic Entity Templates:

Users define templates for Knowledge Base entries (e.g., a "Spell" template with metadata fields like Range, School, Duration).

Entries consist of Template Metadata (for structured UI display/card tags) and a Markdown body (for copy-pasted descriptions).

Typed Counter System: Configurable counters categorized by usage context:

Active/Spendable: High-frequency tracking (e.g., HP, spell slots, ki).

Progression/Passive: Low-frequency tracking updated during pauses (e.g., XP, wealth).

Rest/Recovery Mechanics: Configurable macro actions (e.g., "Short Rest"). Individual counters possess tags defining which macro triggers their reset.

Quick-Access Resource Overlay (HUD):

Triggered via a bottom-to-top swipe gesture.

Auto-populates based on Active/Spendable counters.

Session Notes: Dedicated section for campaign progress with text and image attachments.

Localization:

Primary: English.

Secondary: Ukrainian.

3. Technology Stack

Frontend: TypeScript, React.

Architecture: PWA for mobile-first delivery.

Data Persistence:

Engine: SQLite (via sql.js or wa-sqlite).

Architecture: Unified User DB containing both Character State and the Knowledge Base.

Data Structure: JSON-based Metadata coupled with Markdown strings stored within SQLite rows.

Portability: Exportable .db files for external editing and manual backup.

Media: Avatars and note images stored separately (File System Access API or IndexedDB Blobs).

Styling: Modern Dark UI, soft gradients, high-radius rounded cards.

4. Implementation Details

Knowledge Base Integration: UI supports Markdown rendering and provides a form-builder interface for custom Entity Templates.

UI/UX Reachability Hierarchy:

Tier 0 (Global/Overlay): Resource HUD (Active Counters). Bottom-to-top swipe.

Tier 1 (Primary/Gestures): Combat/Action state, Social state, Session Notes. Horizontal "Swipe-to-Pivot."

Tier 2 (Secondary/Menus): Knowledge Base, Template Editor, Settings, DB Management.

5. Development Roadmap

Phase 1: Knowledge Base Architecture: Design SQLite schema for templates, metadata, and Markdown.

Phase 2: Database Layer: Implement SQLite WASM with export/import functionality.

Phase 3: Markdown & UI Rendering: Build components to render Entity Templates as styled cards.

Phase 4: Navigation & Overlay: Build the gesture HUD (Tier 0) and "Swipe-to-Pivot" interface (Tier 1).

Phase 5: Counter Logic: Implement the counter engine with rest/recovery macro links.