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

---

## Phase 5A: Combat Core Engine (`SM.combat`)

Phase 5A adds the **rules/engine layer** for combat: a reusable, deterministic, testable
combat system with no UI. Phase 5B will build the Battle UI on top of this API. The engine
lives in the `SM.combat` namespace inside `stormmythic.html` and reuses the existing Phase 4
character/derived-stat helpers rather than redefining any formulas.

### Design principles

- **Engine only.** No battle map, HUD, turn-tracker UI, or navigation changes.
- **Deterministic.** All randomness flows through an injectable RNG (`createCombat({ rng })`
  or `createCombat({ seed })`, mulberry32). No uncontrolled `Math.random` in combat paths.
- **Reuse, don't duplicate.** Attack/defense/HP/resource/save numbers come from the existing
  `E.mod`, `E.defenses`, `E.attackBonus`, `E.maxHP`, `E.resource`, `E.saveMod`, `E.profBonus`,
  `E.tierOf`, `E.totalLevel` helpers. Thin compatibility wrappers are exposed under
  `SM.combat.calc` (`getAttributeModifier`, `calculateEvasion`, `calculateBlock`,
  `calculatePhysicalDefense`, `calculateMagicDefense`, `calculateScalingBonus`,
  `calculateAttackPower`).
- **Data safety.** Combat state is fully separate from persistent character records. A
  participant holds a derived *snapshot* plus a `ref` id; the engine never writes to
  localStorage or mutates a stored character.
- **Static hosting.** Pure browser JS. No server, database, env vars, or dependencies.

### Combat state

`createCombat(opts)` returns a plain object: `combatId, round, turnIndex, phase`
(`setup` -> `active` -> `ended`), `participants[]`, `order[]` (initiative order),
`activeId`, `defeated[]`, `log[]` (events), `pendingReactions[]`, `pendingEffects[]`,
`encounter`, `outcome` (`victory`/`defeat`/`null`), and the injected `rng`.

### Participant model

Types: `PLAYER, ALLY, ENEMY, NPC, SUMMON` (sides: party / foe / neutral). Each participant
carries `id, name, type, side, ref, maxHp, hp, tempHp, resources{name,max,current},
defenses{physicalDefense,magicDefense,evasion,block,defenseValue,magicTarget}, attackBonus,
magicAttackBonus, saveMods, speed, initiative(+mod), conditions[], turn (action economy),
reactions[]/passives[], cooldowns, state`. Built from a character via
`snapshotCharacter(char)` or from a generic entity spec (enemy).

### Initiative & turn order

`rollInitiative` (d20 + AGI-based modifier, or supplied/manual values; ties broken by
modifier then stable insertion order). `startCombat`, `startRound`, `startTurn`, `endTurn`,
`getActiveParticipant`, `checkCombatEnd`. Defeated (and downed) participants are skipped;
rounds advance and wrap safely; stalled combat is detected rather than looping.

### Action economy

Per-turn `action`, `bonus`, `reaction`, and `movement` (from speed), reset at the start of
each participant's turn. `spendAction`, `spendBonusAction`, `useReaction` (usable off-turn),
`move`. Invalid/duplicate spends are rejected.

### Attack resolution & attack types

`resolveAttack({ attackerId, targetId, type, bonus?, damage?, ... })`. Types: `melee, ranged,
physical, magical, spell, ability`. Physical uses `defenseValue`; magical/spell uses
`magicTarget`. Returns `{ hit, critical, criticalMiss, attackRoll, attackTotal, defense,
defenseKey, type, damage, damageResult }`. Central `RULES` (nat-20 crit, nat-1 auto-miss).

### Damage & mitigation

Centralized `resolveDamage(...)` returns `{ rawDamage, mitigation, mitigatedDamage,
finalDamage, damageType, critical, tempAbsorbed, targetHpBefore, targetHpAfter, downed,
defeated }`. Critical multiplier, flat typed mitigation (kept separate from the to-hit
defense so it is never double-counted), minimum-damage floor, temporary-HP absorption, then
current-HP reduction. Defenses are consumed through the same pipeline via `getDefenseTarget`.

### Saving throws

`resolveSave({ participantId, attr, dc, bonus? })` -> `{ success, roll, total, dc, attr, mod,
critical }`. Reuses `E.saveMod`; nat-20 auto-success / nat-1 auto-fail (configurable).

### Conditions / status effects

`applyCondition`, `removeCondition`, `hasCondition`, `processConditions(phase)`. Condition
instances track `id, name, duration, permanent, stacks, value, source, tags, startRound`.
Lifecycle: apply / refresh / stack / turn-start & turn-end processing / expiration; cleared
on defeat. Per-tick effects live in the extensible `CONDITION_EFFECTS` registry
(built-ins: Burning, Poisoned, Bleeding, Regeneration) — never in UI code.

### Downed / defeated & healing

State model `healthy | wounded | downed | defeated`. PLAYER/ALLY drop to **downed** at 0 HP
(others are defeated); a downed participant hit again is defeated. Hooks: `rollDeathSave`
(documented default), `revive`. Healing via `resolveHealing(...)` supports HP cap + overheal,
temporary HP (does not stack), and optional revive.

### Resources

`checkResource`, `spendResource`, `restoreResource` operate on each participant's
`{name,max,current}` pool (from `E.resource`), rejecting insufficient spends.

### Ability / spell hooks (for Phase 5C)

`executeAbility(...)` and `castSpell(...)` provide the central execution pathway: validate
actor, check resources *before* consuming the action, spend action + resource, then dispatch
to a registered handler (`ABILITY_HANDLERS` / `SPELL_HANDLERS`). Handlers call the same
`resolveAttack`/`resolveDamage`/`applyCondition` pipeline — no rule duplication. Phase 5C
fills in the content.

### Reaction / passive hooks & events

Event-driven, not a hardcoded list. Participants register `reactions[]`/`passives[]`
(`registerReaction`, `registerPassive`); matching events queue into `pendingReactions`.
`emitCombatEvent`, `on(state, type, fn)`, `getCombatLog`. Structured event types:
`COMBAT_STARTED, ROUND_STARTED, TURN_STARTED, TURN_ENDED, ATTACK_DECLARED, ATTACK_RESOLVED,
DAMAGE_DEALT, HEALING_APPLIED, CONDITION_APPLIED, CONDITION_REMOVED, RESOURCE_SPENT,
RESOURCE_RESTORED, PARTICIPANT_DOWNED, PARTICIPANT_DEFEATED, SAVE_RESOLVED, ABILITY_EXECUTED,
SPELL_CAST, REACTION_TRIGGERED, COMBAT_VICTORY, COMBAT_DEFEAT`. Events carry structured data;
presentation is left to the UI layer.

### Public API (selected)

`createCombat, addParticipant, removeParticipant, getParticipant, snapshotCharacter,
rollInitiative, startCombat, startRound, startTurn, endTurn, getActiveParticipant,
spendAction, spendBonusAction, useReaction, move, resolveAttack, resolveDamage,
resolveHealing, resolveSave, applyCondition, removeCondition, processConditions,
checkResource, spendResource, restoreResource, executeAbility, castSpell, registerReaction,
registerPassive, getPendingReactions, revive, rollDeathSave, checkCombatEnd, emitCombatEvent,
on, getCombatLog, makeRng, RULES, EVENTS, calc`.

### Extension points for Phases 5B–5E

- **5B (Battle UI):** read `state.log`/`getCombatLog` for the combat log, drive the turn
  tracker from `order`/`activeId`, render participants and action economy from the
  participant model.
- **5C (Abilities/Spells):** register `ABILITY_HANDLERS` / `SPELL_HANDLERS`.
- **5D/5E (AI, encounters, rewards, multiplayer):** listen via `on(...)`, resolve queued
  `pendingReactions`, and build encounters/rewards around `checkCombatEnd`.

Central tunables live in `SM.combat.RULES` (crit rules, minimum damage, wounded threshold,
downed-state types) so balance can change in one place.
