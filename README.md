# PF2e Party Bestiary

A Bestiary tab on every player's character sheet. Creatures the party has
met show up redacted, and facts unlock as the GM reveals them, a player
researches them in downtime, or the party studies a dragged-in stat block.

## Install

**The easy way (Windows):** download the [SpazzMods Installer](https://github.com/Spazzletopia-Studios/spazzmods-installer/releases/latest),
run it, and click Install on PF2e Party Bestiary. No account needed.

**Without the installer:** paste this into Foundry's **Install Module →
Manifest URL** box:
`https://github.com/Spazzletopia-Studios/pf2e-party-bestiary/releases/latest/download/module.json`

## Using it

- Look for the paw-icon **Bestiary** tab on any player character sheet.
- In play, control one character, target one creature, and click **Recall
  Knowledge** in Token Controls or the **SpazzMods** selector. Pick a first and
  second knowledge priority. A success reveals the first, and a critical
  success reveals both. A failure changes nothing. A critical failure records
  one believable false lead, which a later successful check corrects. The
  creature portrait flashes across the screen, then rises and remains above
  the learned information as its panel appears. The action works even when the
  creature is not in the Bestiary yet.
- The module creates **Party Bestiary — Recall Knowledge** in the Macro
  Directory. Drag that macro to any hotbar slot for the same action.
- Each known creature is a collapsed card — portrait, name, level, a one-line
  teaser. Click to expand, or search by name.
- Unknown facts show redacted with a **Research** button — it rolls the
  creature's identification skill against its standard DC. A critical
  success unlocks one extra fact.
- Add a creature by typing a few letters to search compendiums, or drag one
  in from a compendium or the Actors tab — it arrives fully locked until
  someone learns something about it.
- The GM can **Reveal** any fact directly, write table notes on a creature,
  or remove entries.
- Install **PF2e Encounter Console** too (0.23.0+) and the two work together
  automatically — nothing extra to set up.

---

**Foundry v13–v14 · PF2e system 8.x** (built and verified against PF2e 8.4.0 / Foundry 14.363)

This tab used to ship inside **PF2e Encounter Console**. It is its own module
now so a table can have it without the console. With the console installed
(0.23.0 or newer) the two work as one — see *With the Encounter Console*.

Upgrading from an Encounter Console that had the tab built in: enable this
module and load the world as GM once. Everything the party knew — the shared
store, each character's private notes, the shared/personal switch — moves
over by itself. The old data is left in place, untouched.

## What it does

- A **Bestiary** tab (paw icon) on each player character sheet
- A standalone **Recall Knowledge** scene action. It uses one secret d20 for
  every relevant identification skill and the character's Lore skills, then
  uses the best applicable standard skill for the automatic result and whispers
  the full PF2e DC-progression card to the GM. Lore remains on that private card
  for GM judgment because the system cannot know whether a custom Lore is broad,
  specific, or relevant. It does not call PF2e Workbench, PF2e HUD, or any other
  module. Critical-failure false leads look like ordinary facts to players; the
  GM sees a marker on the Bestiary row, and later true knowledge replaces them.
- Every creature is a collapsed card: portrait, name, level, progress bar, a
  one-line teaser of what is known. Expand one or all; search by name.
- A known **Type** line shows as chips — rarity, size, traits — with the same
  glossary tooltips the NPC sheet has
- Unknown lines are redacted with a **Research** button
- **Research** rolls the creature's identification skill against a flat DC
  (its standard identification DC; no escalation — downtime is paid in time,
  not in a rising DC). Items that help — a Scholarly Journal, anything whose
  text mentions Recall Knowledge — are offered, never applied silently. A
  critical success lets the player claim one more fact from the card.
- **Add creatures**: type three letters to search every Actor compendium, or
  drag a creature from a compendium or the Actors tab onto the panel. It
  arrives fully locked — dragging a stat block in is not knowing it.
- **Notes** pane per creature: the stat block's flavour text (once learned),
  plus the GM's own **Table notes** — campaign lore, a hook, homebrew rules —
  shared with the players or kept GM-only
- **GM controls**: **Reveal** a line outright (the table was just told — it is
  recorded for the party and posted to chat), remove a line or a whole
  creature, write table notes, and **Sync all bestiaries** to pool every
  character's private notes into the party store

## Settings

| Setting | Default | Notes |
|---|---|---|
| Bestiary knowledge is shared | on | On: the party pools what it learns; every character's tab shows it. Off: each character sees only what they personally learned or researched — for tables that prefer immersion. |

## With the Encounter Console

Install **PF2e Encounter Console 0.23.0+** alongside and the console's **Know**
tab writes to this module's store: every Recall Knowledge fact the GM posts
from the console unlocks on the players' Bestiary tabs, every creature the
console has looked at gets locked slots on the sheets, and a player's downtime
research shows as ticked on the Know tab. Nothing to configure.

Without the console, the GM reveals facts with the **Reveal** button on any
character's Bestiary tab.

## API

Other modules reach the store through `game.modules.get("pf2e-party-bestiary").api`
(also `globalThis.pf2ePartyBestiary`). The surface the console uses:

| Call | What it does |
|---|---|
| `creatureKey(actor, tokenDoc?)` | A key stable across every copy of a creature (compendium source, dots stripped) |
| `slimFacts(facts)` | The storable shape of a fact list — tags and prose included |
| `partyAll()` | Every creature the party has knowledge of, keyed by creature |
| `partyUpdate(key, mutate)` | GM-only write to one creature's record; `attempts` and `revealed` are always present |
| `rememberCreature(key, meta)` | Record what a creature *is* so the sheets can show locked slots; a no-op unless something changed |
| `bestiaryFor(actor)` | The merged view a sheet renders for one character |
| `revealFact(actor, key, factKey, entry)` | A player learned a fact — personal tier, and the party tier via the GM |
| `addFromActor(characterActor, npcActor, facts)` | Add a creature, fully locked |
| `applyRecallResult(characterActor, key, result)` | Apply one completed Recall result, including reveals, false leads, and attempt count |
| `recallKnowledge()` | Run the independent in-play action from the controlled character and current target |
| `ensureRecallMacro()` | Create or reuse the shared draggable Recall Knowledge macro (GM only) |

The module creates this macro for you. Its command is:

```js
game.pf2ePartyBestiary.recallKnowledge({ event });
```

The store announces every write with `Hooks.callAll("pf2e-party-bestiary.knowledgeChanged", store)`.

## Architecture

- `scripts/knowledge.js` — the store. Two tiers: **party** (world setting,
  GM-written; players' writes go through a socket request the one active GM
  services) and **personal** (a flag on the character's own actor, which the
  player can write). The sheet merges them according to the setting above.
- `scripts/bestiary.js` — the tab: sheet injection, rendering, research,
  search, drag-in, GM controls.
- `scripts/extract.js` — the only file that touches PF2e's data shape. Shared
  **by copy** with the Encounter Console (the console owns it); see `agents.md`.
- `styles/bestiary.css` — scoped under `.ec-bestiary` / `.ec-research`.

## Testing

```
cd harness && npm i
node harness/smoke-test.mjs       # the store, migration, rendering, research — headless
node harness/browser-test.mjs     # the tab injected into a stand-in sheet, in a real browser
```

`browser-test.mjs` exists because the tab is injected into **another
package's DOM** and its visibility is decided by CSS. A blank panel, a panel
bleeding over other tabs, or a sheet left empty after switching away are all
invisible to a logic test — every one of those actually shipped. It runs the
real `bestiary.js` against a stand-in sheet and asserts computed styles.

Worth running before every install. `node --check` passes files that the ESM
loader then rejects, and a half-applied edit produces a module that loads but
silently does nothing.

## Afflictions

With the Spell Smoothing affliction tracker installed (optional), every poison,
disease, or curse the party identifies is recorded here too. The Bestiary tab gains
**Creatures | Poisons | Diseases | Curses** tabs, always present (an empty kind
shows an ominous note), with the same cards, research, and GM reveals as creatures.
On identification the party learns the kind, level, save DC, onset, maximum
duration, and the stage they are at; each further stage unlocks as it is
suffered. The other way round, revealing a creature's venom ability here
identifies open cases in the tracker. Neither module requires the other.
