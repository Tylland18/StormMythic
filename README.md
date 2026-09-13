# StormMythic

A complete, offline-capable dark-fantasy tabletop RPG platform delivered as a **single self-contained HTML file** (`stormmythic.html`). Original IP only. Open the file in any modern browser — no server, no build step, localStorage persistence, mobile-first with a desktop sidebar.

## Contents

- **`stormmythic.html`** — the entire app (engine, data registries, UI, and world lore) in one file.
- **`data/lore/`** — the modular world-lore source, kept separate from UI. These JSON files are the human-editable source for the lore that is bundled into `stormmythic.html`.

## World: Aethyr, the Storm-Wracked World

A continent of survivor-cultures raised in the ruin of a vanished golden age, where the broken sky let the living Storm into the world.

- **21 Kin** and **138 Lineages**, each with world lore (homeland, culture, region, traditions, appearance, occupations, allies/rivals, landmarks, and story hooks).
- **15 major regions**, a **9-age history timeline**, **18 factions**, and **26 named settlements**.
- Lore is **cosmetic and separate from mechanics**: any Kin or Lineage can become any Class or Discipline. No lineage locks a character into a build.

## Lore data files (`data/lore/`)

- `world.json` — regions, history ages, factions, settlements, world meta.
- `kin.json` — per-Kin lore (21), keyed by stable kin id.
- `lineages/*.json` — per-Lineage lore (138), keyed by stable lineage id.
- `birdkin_new.json` — mechanical records for the 20 catalog Birdkin lineages.

## In-app navigation

Character Creation, Character Sheet, Character Forge, Combat, Inventory, Equipment, Dice Roller, the Kin / Lineage / Class / Discipline / Ability / Spell libraries, the **Item Codex** (1,460 items), and the **World Codex** (regions, history, factions, settlements). Selecting a Kin and Lineage in the creator shows full world lore alongside the mechanical traits.
