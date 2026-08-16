# GATEWAY — Single-File HTML5 Dungeon Crawler
## Recreation Specification / Plan Document

This document is a complete, self-contained specification of the game in
`apshai.html`. It is written so that a competent engineer **or another LLM**
can rebuild the same game and functionality from this document alone, without
seeing the source. Where exact values matter (colors, timings, stats, pixel
art, font glyphs), they are given verbatim. Where a system is described, the
algorithm is given precisely enough to reproduce behavior.

---

## Known Issues & Roadmap (at a glance)

**Done in v1.1** (icon/sprite pass, §21.1):
- **Door** → framed doorway (dark opening, wood frame, two planks, keyhole).
- **Bow** → distinct curved stave + string + nocked arrow (no longer a sword).
- **Pickup icons** → each unique: chest, flask, key, quiver, sword, bow, and strength (diamond) / agility (chevron) / luck (clover) glyphs.
- **Minimap** → corner overview, toggle **K** (§15.11).

**Done in v1.2** (high-impact balance/look wins):
- **Melee reach** widened (`<20` → `<24`) so you don't have to overlap monsters.
- **Crits** — luck-driven 2× hits shown as a bigger white damage number.
- **Agility** now scales speed more (`+0.05/pt`, was `+0.03`); monster speeds tightened (goblin 50, orc 45).
- **Item density** up (`12 + level*2`, was `10 + level*2`).
- **Look:** floor checkerboard + speckle, wall top-edge highlight, and a backing plate behind every item so they pop.

**Done in v1.3** (bug fixes):
- **Inventory carry-over** — weapons, arrows, keys, potions, and stats now persist across dungeons (you no longer lose your loadout). HP refills to max on each new dungeon.
- **Dungeon solvability** — doors are now placed with a BFS reachability check so the exit is always reachable; each door is guaranteed a key.
- **No more door walls** — doors are kept apart (no two adjacent), so you never get a corridor full of doors.
- **Keys never spawn in walls** — every key (door keys and `mkItem` keys) is repositioned onto a clean floor tile after placement.
- **Removed boxy sprite outlines** (looked like boxes around hero/monsters).

**Done in v1.4** (authentic mechanics port, inspired by the original's RE docs):
- **Combat formulas** — melee damage = weapon power + strength/4 (+3 vs undead if Cross) − monster armor; bow arrows = 4, magic arrows = 8, both − armor. Monster attacks use a hit-chance roll (`1..31 > agility` to hit) and damage = monster weapon power − armor − shield (− level on a lucky roll).
- **15 monster types** (level-gated) with hp/armor/weapon power and AI (walk / confuse / berzerk). 8 monsters per dungeon: 2 of each current-level type + random from ≤ level.
- **Equipment** — 4 weapons (dagger→2-hand), 4 armors, 2 shields, Cross (+3 vs undead), Wand. Armor/shield reduce incoming damage. All carry over between dungeons.
- **Item distribution** — each dungeon: guaranteed arrows + (level+2) random level items + 16 treasures with a fixed quality mix (3 each lead/iron/bronze/silver/gold, 1 jeweled). Gold (50%) and jeweled (100%) treasures can be trapped.
- **Traps** — level-gated (pit, stone, freeze, death), `level+2` per dungeon (max 16). Freeze immobilizes the player; damage traps use the monster-attack formula.
- **New sprites** — rat, slime, bat, skeleton (ghoul/mummy), dragon; other types get a tinted blob. New item icons for armor, shield, cross, magic arrows, wand, dagger, 2-hand sword.

**Done in v1.5** (spells, wand, boss, fog-of-war, bug fixes):
- **Spells** — 6 spell types found as items in dungeons. Finding one increments `p.spells[type]` (a count). Casting from the **Spells** submenu (M → Spells) consumes one charge. **Stun** (freezes all monsters for `(luck+agi+wand)/2` U), **Confuse** (visible monsters wander permanently), **Map** (reveals the whole dungeon), **Protect** (disables traps for `(luck+wand)*10` U), **Teleport** (jump to a random floor tile ≥25% from the border), **Blast** (tunnels 3 tiles in the facing direction). Durations use the original's unit `U = 0.5s`. The menu shows only collected spells with counts (e.g. "Stun Spell x2").
- **Wand** — `p.wand` scales stun/protect durations (and is reserved for future magic-item scaling).
- **Boss** — each level's final dungeon (D16) spawns a level-specific boss (Giant→Dragon) near the exit that always chases. A boss health bar (name + red bar) shows in the bottom-left while the boss is alive. Bosses have their own hp/weapon/armor/radius and a distinct golden-crowned sprite.
- **Walk animation** — hero and monsters bob 1px up on alternate frames while moving (~6 bobs/s, `Math.floor(anim)%2`); each monster starts with a random phase so bobs are out of sync. Frozen/idle entities stay still.
- **Fog of war** — the minimap now only shows explored tiles. `revealAround(tx,ty,3)` in `updatePlayer` reveals a 3-tile radius as you walk; the **Map** spell reveals the entire dungeon.
- **Bug fixes:** menu M-key toggle (M now closes the menu, not just opens it); Esc closes menus before toggling pause; spell menu shows collected spells only (not all 6); spell counts instead of duplicate rows; weapon pickup deduplication (picking up a weapon you already have shows "Already have" and removes the item from the map); HUD arrow count only shows when the bow is equipped (no overlap with weapon name); status panel footer spacing fixed.
- Spell-effect timers (`stunT`, `confuseT`, `protectT`) and spell counts carry over between dungeons.

**Done in v1.6** (audio, §22):
- **Procedural WebAudio chiptune** — no audio assets; everything synthesized (oscillators + noise buffers).
- **15 SFX** — swing, shoot, hit, kill, hurt, die, pickup, treasure, door, trap, spell, select, move, levelup, boss intro.
- **Music** — step-sequenced 8-bar loops (square lead + triangle bass, SID-style): a 110 BPM dungeon theme and a 132 BPM boss variation (sawtooth lead, denser bass). Lookahead scheduler (25 ms tick, 120 ms horizon) with tab-throttle resync.
- **Autoplay policy** — `AudioContext` created on first keydown and resumed if suspended. **Z** toggles mute (master gain 0 ↔ 0.15); a red `MUTED (Z)` indicator shows bottom-right while muted.
- Boss music auto-starts when a boss spawns and reverts to the dungeon theme when the boss dies or the dungeon advances.
- **Spell-menu hardening:** spell-index clamped after a cast shrinks the list (no stale-index crash), submenu auto-closes when the inventory empties (no NaN from modulo-by-zero), and spell pickup messages now say `… found! (M to cast)` so it's clear spells are inventory items that must be cast, not instant effects.

**Top roadmap items** (details in §21.2): high-score persistence (`localStorage`), touch controls, and the remaining balance/look polish (see §21.2).

> **Status:** v1.6 is complete and playable (see §19 for how it was verified). The items above are the planned next steps.

---

## 1. Purpose & Scope

**What it is:** A top-down, real-time, scrolling dungeon crawler in the spirit
of the 1983 C64 game *Gateway to Apshai*. It is an **original tribute**: the
name, sprites, text, maps, and layout are original. Only *genre conventions*
are shared (three stats, treasure = material value × level, a per-level timer,
action menus, keys/doors/traps/secrets). No copyrighted assets are copied.

**Deliverable:** A single self-contained file `apshai.html` (HTML + CSS + JS
inline). No build step, no external libraries, no external assets, no network
requests. It runs by opening the file in any modern browser (double-click, or
`file://`).

**Feel:** Pixel-art, C64-style palette, crisp 5×7 bitmap font, 320×200 logical
resolution upscaled with nearest-neighbor.

---

## 2. Constraints & Non-Goals

- **Single file.** Everything (style, script, art, font) is inline.
- **No dependencies.** Vanilla JS only. No frameworks, no fetch, no CDN.
- **No audio** in v1 (a natural future extension, see §21).
- **Deterministic dungeons** via a seeded PRNG (same level+dungeon always
  yields the same map).
- **Original content** — do not reuse the 1983 game's name, sprites, music, or
  text.

---

## 3. Technical Foundation

| Item | Value |
|---|---|
| Logical canvas size | `W = 320`, `H = 200` |
| Tile size | `TILE = 16` px |
| Map size | `MAP_W = 48`, `MAP_H = 34` tiles (768×544 px world) |
| HUD height | top `12` px |
| Canvas element | `<canvas id="game" width="320" height="200">` |
| Smoothing | `ctx.imageSmoothingEnabled = false` |
| CSS upscaling | `image-rendering: pixelated` (and `crisp-edges`) |
| Script mode | `"use strict"` |
| Frame loop | `requestAnimationFrame`; `dt = (now-last)/1000`, clamped to `0.05` |

**Coordinate system.** The world is measured in pixels. A tile `(tx,ty)`
occupies pixels `[tx*16, tx*16+16) × [ty*16, ty*16+16)`. Entities store a
pixel-center `(x,y)`. The tile under an entity is `(floor(x/16), floor(y/16))`.

**Camera.** The 320×200 viewport follows the player, centered, clamped so it
never shows outside the map:
```
camX = clamp(round(p.x - W/2), 0, MAP_W*TILE - W)
camY = clamp(round(p.y - H/2), 0, MAP_H*TILE - H)
```
Everything in the playfield is drawn at `(worldX - camX, worldY - camY)`.

**Page layout.** Centered column: the canvas, then a small `#hint` line under
it listing the controls. Body background `#0a0a12`, text `#cfd2ff`,
monospace font. Canvas has a `2px solid #2a2a44` border and
`aspect-ratio: 320/200`, width `min(92vw, 92vh*1.6)`.

**Main loop** (runs always, regardless of state):
```
loop(now):
  dt = min((now-last)/1000, 0.05); last = now
  update(dt)
  render()
  clearJust()          # clear one-press key flags
  requestAnimationFrame(loop)
```

---

## 4. Visual System

### 4.1 Palette (exact RGB)

A 16-color C64-ish palette. `rgb(a)` builds `'rgb(r,g,b)'`.

| Name | RGB | Name | RGB |
|---|---|---|---|
| `black` | `[0,0,0]` | `orange` | `[220,140,60]` |
| `white` | `[220,220,220]` | `brown` | `[150,100,50]` |
| `red` | `[200,50,50]` | `lred` | `[230,120,120]` |
| `cyan` | `[80,200,220]` | `dgray` | `[80,80,92]` |
| `purple` | `[160,80,200]` | `gray` | `[150,150,160]` |
| `green` | `[60,200,80]` | `lgreen` | `[140,220,120]` |
| `blue` | `[50,80,200]` | `lblue` | `[120,160,235]` |
| `yellow` | `[220,200,60]` | `lgray` | `[200,200,205]` |

### 4.2 Bitmap font (5×7)

**Why a bitmap font:** drawing text with the browser's system font and then
upscaling produces anti-aliased blur. Instead every glyph is a 5-wide × 7-tall
grid of `0`/`1` strings, drawn with integer `fillRect` — hard-edged, no
anti-aliasing. This is the single most important detail for the crisp look.

**Glyph set:** space, `! " ' ( ) * + , - . / : ; ? [ ]`, digits `0-9`,
uppercase `A-Z`, lowercase `a-z`.

**Text helpers:**
- `text(str, x, y, color, scale=1)` — draws left-anchored at `(x,y)`. For each
  character, for each `1` bit at row `r`, col `c`, fill a `scale×scale` rect at
  `(x + c*scale, y + r*scale)`. Advance `x` by `6*scale` per character
  (5px glyph + 1px spacing).
- `textW(str, scale=1)` — width = `len*6*scale - scale` (drop the trailing
  1px gap).
- `textC(str, cx, y, color, scale=1)` — centered: `text(str, round(cx - textW/2), y, ...)`.
- `textOutlined(str, x, y, color, scale=1)` — draws the string in black at
  `(x±1, y)` and `(x, y±1)` (4 offsets) then the real color on top. Used for
  damage numbers so they read on any tile.

The **full glyph table** is in **Appendix A**.

### 4.3 Pixel sprites

Sprites are small ASCII pixel maps. Each character maps to a color via
`LEGEND`; `.` is transparent. `makeSprite(rows)` renders the map to an offscreen
canvas (1px per cell).

**LEGEND (exact RGB):**

| ch | RGB | ch | RGB | ch | RGB |
|---|---|---|---|---|---|
| `.` | (none) | `g` | `[70,190,90]` | `n` | `[140,95,50]` |
| `k` | `[12,12,14]` | `b` | `[60,110,220]` | `R` | `[230,120,120]` |
| `w` | `[235,235,235]` | `y` | `[230,210,70]` | `d` | `[80,80,92]` |
| `r` | `[200,50,50]` | `o` | `[222,164,112]` | `s` | `[172,172,182]` |
| `c` | `[80,200,220]` | `p` | `[160,80,200]` | `G` | `[150,220,130]` |
|   |   |   |   | `B` | `[130,170,235]` |
|   |   |   |   | `S` | `[212,212,218]` |

**Sprite sizes:** `hero` = 12×14, `goblin` = 12×12, `orc` = 14×14.
The exact pixel maps are in **Appendix B**.

`drawSprite(img, x, y, facing)` centers the image on `(x,y)` and mirrors it
horizontally when `facing === 'left'`.

---

## 5. Core Data & Constants

### 5.1 Tile types
```
T = { WALL:0, FLOOR:1, DOOR:2, TRAP:3, SECRET:4, EXIT:5 }
```
Solid (blocks movement) = `WALL` and `DOOR`. `TRAP`, `SECRET`, `EXIT` are
floor-like (walkable) with special behavior.

### 5.2 Weapons & Equipment (v1.4)
```
WEAPONS = {
  dagger:     { name:'Dagger',        wp:1 },
  shortsword: { name:'Short Sword',   wp:2 },
  longsword:  { name:'Long Sword',    wp:3 },
  twohand:    { name:"2-Hand Sword",  wp:4 },
  bow:        { name:'Bow',           wp:0 }   # ranged; consumes arrows
}
ARMORS  = { leather:{ap:1}, chain:{ap:2}, breast:{ap:3}, plate:{ap:4} }
SHIELDS = { small:{sp:1}, large:{sp:2} }
```
- `wp` = weapon power (melee base). `ap` = armor power (reduces incoming damage).
- `sp` = shield power (reduces incoming damage). Player starts with `leather`
  armor and no shield.
- **Cross** (`p.cross`): +3 melee damage vs undead monsters.
- **Wand** (`p.wand`): reserved for future magic scaling.
- **Bow**: normal arrows deal 4, magic arrows deal 8 (both − monster armor).
  Magic arrows take priority when both are held.

### 5.3 Treasure materials & kinds
```
MATS = [
  { name:'Lead',    v:0,    col: gray  },
  { name:'Iron',    v:10,   col: lgray },
  { name:'Bronze',  v:25,   col: orange},
  { name:'Silver',  v:50,   col: white },
  { name:'Gold',    v:100,  col: yellow},
  { name:'Jeweled', v:1000, col: purple},
]
TREAS = ['Necklace','Chest','Coffer','Chalice','Sceptre','Crown','Plaque']
```
**Score for a treasure = `material.v * currentLevel`.** (This is the core
scoring rule of the original genre.)

### 5.4 Player defaults (`newPlayer()`) — v1.5
```
{ x:0, y:0, r:5, facing:'down', anim:0, moving:false,
  str:3, agi:3, luck:3, hp:9, lives:5,
  weapons:['dagger'], weaponIdx:0, arrows:0, magicArrows:0, keys:0, potions:0,
  spells:{},
  armor:'leather', shield:null, cross:false, wand:0, freezeT:0,
  stunT:0, confuseT:0, protectT:0,
  attackCd:0, swing:0, invuln:0 }
```
- `maxHp() = str + agi + luck` (starts at 9).
- `weapon() = WEAPONS[p.weapons[p.weaponIdx]]`.
- `spells` is a count object (e.g. `{stun:2, map:1}`); empty `{}` at start.
- Movement speed = `90 * (1 + agi*0.05)` px/s.
- `freezeT` > 0 → player cannot move or attack (freeze trap).
- `stunT`/`confuseT`/`protectT` > 0 → active spell effects (§11.5).

### 5.5 Monsters (v1.4)
15 level-gated types in `MONSTERS`, each with `hp`, `wp` (weapon power),
`armor`, `spd`, `ai` (`walk`/`confuse`/`berzerk`), and `undead` flag:
```
rat(1) snake(1) fungus(1) bat(1)          # level 1
caverat(2) slime(2) spider(2)             # level 2
ghoul(3,undead) ogre(3) goblin(3)         # level 3
mummy(4,undead) priest(4)                 # level 4
vampire(6,undead)                         # level 6
warrior(7)                                # level 7
dragon(8)                                 # level 8
```
`mkMonster(type,x,y,lv)` builds a monster from its `MONSTERS` entry.
`pickMonsterTypes(lv,rng)` returns 8 types: 2 of each type in the current
level, the rest random from ≤ current level.
*(v1.4: replaced the old goblin/orc pair with this full roster.)*
*(v1.2: goblin `spd 55→50`, orc `spd 40→45` — tighter speed band.)*
`aggro` = radius to start chasing; `lose` = radius to give up; `hitT` =
white-flash timer after being hit; `attackCd` = cooldown between bites.

### 5.6 Seeded RNG
`mulberry32(a)` — the standard 32-bit PRNG:
```
function mulberry32(a){
  return function(){
    a|=0; a = a + 0x6D2B79F5 | 0;
    let t = Math.imul(a ^ a>>>15, 1|a);
    t = t + Math.imul(t ^ t>>>7, 61|t) ^ t;
    return ((t ^ t>>>14) >>> 0) / 4294967296;
  };
}
```
**Dungeon seed** = `level*7919 + dungeon*104729 + 12345`. One RNG instance per
dungeon drives all generation (rooms, corridors, placement, item rolls).

---

## 6. Game State & State Machine

### 6.1 State variables
```
p, monsters, items, projectiles, dmgNums      # dmgNums = floating damage numbers
level, dungeon, score, timeLeft, titleDungeon=0
state='title', flash=0, flashT=0, msgQ=[], panel=null,
menu=false, menuIdx=0, paused=false, tGlobal=0, minimap=true
```
- `state` ∈ { `'title'`, `'play'`, `'over'` }.
- `paused` (boolean) — toggled by Esc during play.
- `panel` — when non-null, a status/info panel is shown (game logic frozen).
- `menu` (boolean) — the action menu overlay (game logic frozen).
- `flash` ∈ {0, 1, 2}; `flashT` = remaining flash time.
  `1` = green (you hit something), `2` = red (you were hit).
- `msgQ` — queue of `{text, t}` floating messages (each lives 2.2s, max 4).
- `tGlobal` — global clock (for pulsing/blink effects).
- `minimap` (boolean) — show the corner minimap; toggled by **K** during play.
  Reset to `true` in `newLevel()` (§15.11).

### 6.2 Transitions
- `title` → `play`: Enter/Space (starts at chosen dungeon, level 1).
- `play` → `over`: lives reach 0.
- `over` → `title`: Enter/Space.
- `play` ↔ paused: Esc.
- `play` with `panel`/`menu`: logic frozen, overlay shown; dismiss with
  M/Enter/Space/Esc.

---

## 7. Dungeon Generation

### 7.1 `genDungeon(level, dungeon)`
1. Create `rng = mulberry32(seed)`.
2. Fill `map` (Uint8Array, `MAP_W*MAP_H`) with `WALL`.
3. **Rooms.** Target count = `9 + floor(rng()*4)` (i.e. 9–12). Repeatedly
   (guard ≤ 400 tries) propose a room:
   - `rw = 4 + floor(rng()*6)` (4–9), `rh = 3 + floor(rng()*5)` (3–7).
   - `rx = 1 + floor(rng()*(MAP_W-rw-2))`, `ry = 1 + floor(rng()*(MAP_H-rh-2))`.
   - Reject if it overlaps any existing room **with a 1-tile margin**:
     overlap if `rx < o.x+o.w+1 && rx+rw+1 > o.x && ry < o.y+o.h+1 && ry+rh+1 > o.y`.
   - Accept: store `{x,y,w,h,cx:x+(w>>1),cy:y+(h>>1)}`; set its interior to `FLOOR`.
4. **Corridors.** Connect room `i-1` to room `i` with an **L-shaped** corridor
   (1 tile wide), carved through centers. 50/50: horizontal-first then
   vertical, or vertical-first then horizontal. `carve(x,y)` sets `FLOOR`
   (ignoring the outer border).
5. Allocate `trapDone` and `revealed` (Uint8Array, all 0).
6. Call `placeEntities(level, dungeon, rng)`.

### 7.2 `placeEntities(level, dungeon, rng)`
- **Player** at room 0 center: `p = newPlayer(); p.x=(cx+0.5)*16; p.y=(cy+0.5)*16`.
  `newPlayer()` copies inventory/stats from `window.__carry` if set (v1.3), so
  weapons/arrows/keys/potions/stats persist across dungeons; HP refills to max.
- **Exit** at the **last** room's center tile → set that tile to `EXIT`.
- **Gate exclusion** (v1.6): a `spotIn(room)` helper picks a random interior
  tile of a room and retries (up to 8×, then nudges +1 tile) if it lands on the
  gate tile. All monsters, treasures, and level items use it, so **nothing
  spawns on the gate**. (Traps, secrets, and keys already can't land there —
  they require a `FLOOR` tile and the gate is `EXIT`.)
- **Monsters** (v1.4). Exactly 8 per dungeon: 2 of each current-level type +
  random types from ≤ level (`pickMonsterTypes`). Each placed via `spotIn` in a
  random room (repeats allowed, room 0 allowed).
- **Boss** (v1.5): in dungeon 16 of each level, a level-specific boss spawns
  adjacent to the exit (intentionally *near* the gate, not on it).
- **Items.** 16 treasures with a fixed quality mix (`mkTreasures`: 3 each of
  lead/iron/bronze/silver/gold, 1 jeweled) + guaranteed arrows + `level+2`
  random level items from `mkItem` (§7.3). All placed via `spotIn` in a random
  room.
- **Doors** (up to 4, solvable, clustered — v1.3). Collect all corridor-neck
  `FLOOR` tiles (left+right floor & up+down not, **or** up+down floor &
  left+right not), shuffle them, then for each candidate: skip it if it's
  adjacent to a door already placed (so doors never line up), tentatively set
  it to `DOOR`, and run a BFS from the start room treating doors as walls; if
  the exit is **still reachable**, keep the door, otherwise revert it to
  `FLOOR`. After all doors are placed, add one `key` item per placed door on a
  clean `FLOOR` tile (retry until a floor tile is found), so every door is
  guaranteed a key and the exit is always reachable.
- **Key repositioning** (v1.3): after *all* items/traps/secrets are placed, any
  key that landed on a non-floor tile (wall, trap, secret, or exit) is moved to
  a clean `FLOOR` tile. This covers keys generated by `mkItem` as well as the
  door keys.
- **Traps** (up to 8). Random `FLOOR` tiles (not the player's start tile) →
  `TRAP`.
- **Secrets** (up to 6). Random `FLOOR` tiles that are **adjacent to at least
  one wall** → `SECRET` (hidden floor that can be searched).

`pickRoom(rng)` returns room 0 if only one room, else a random room from index
1..end (never room 0).

### 7.3 `mkItem(x, y, rng, lv)` — item roll
```
roll = rng()
<0.50  treasure:  mi = min(5, floor(rng()*3 + lv*0.6)); mat=MATS[mi]
                    → {type:'treasure', mat:mat.name, value:mat.v,
                       kind:TREAS[floor(rng()*len)], col:mat.col}
<0.62  potion
<0.70  arrows:    n = 6 + floor(rng()*8)   (6–13)
<0.76  key
<0.82  shortsword
<0.87  longsword
<0.90  bow
<0.94  strength
<0.97  agility
else   luck
```
Higher levels bias treasure toward more valuable materials (`lv*0.6` term).

### 7.4 `newLevel(lv, dn)` / `startGame()`
- `newLevel`: set `level,dungeon`; `genDungeon`; reset `timeLeft=390`,
  `projectiles=[]`, `msgQ=[]`, `dmgNums=[]`, `panel=null`, `menu=false`,
  `menuIdx=0`.
- `startGame`: `score=0`; `window.__carry=null`; `p=newPlayer()`; `newLevel(1,1)`;
  `state='play'`.

---

## 8. Player

### 8.1 Movement & collision
`updatePlayer(dt)`:
- Read direction from held keys (Arrows or WASD). If diagonal, multiply both
  components by `0.7071`.
- `spd = 90*(1+agi*0.05)`.
- Set `facing` from the dominant axis (`left/right` if dx, else `up/down` if dy).
- `moveEntity(p, dx*spd*dt, dy*spd*dt)`.
- Decrement `attackCd`, `swing`, `invuln`.
- **Tile effects** at the player's tile (`tx,ty`):
  - `TRAP` and not yet triggered → mark `trapDone` and `revealed`,
    `damagePlayer(1)`, message `Trap!`.
  - `EXIT` → `advanceLevel('exit')` and return.
- **Door opening:** if the tile the player *faces* is a `DOOR` and the player
  has a key and is within `1.6*TILE` of the door center → consume a key, set the
  tile to `FLOOR`, message `Door opened`.
- **Attack:** if `just['Space']` or `just['KeyX']` → `tryAttack()`.

**Collision** — `moveEntity(e,dx,dy)` moves axis-by-axis and reverts an axis if
it collides (allows sliding along walls). `collides(e)` checks every tile
overlapping the entity's radius box `[x-r, x+r] × [y-r, y+r]` for a solid tile.

### 8.2 Attack (`tryAttack`) — v1.4
- If `attackCd>0` **or** `freezeT>0`, ignore. Else set `attackCd=0.32`, `swing=0.16`.
- **Bow** (current weapon is `bow`): if `magicArrows>0` consume one and spawn a
  projectile with `dmg:8, magic:true`; else if `arrows>0` consume one with
  `dmg:4, magic:false`. Projectile: `{x,y, dx:dirX*230, dy:dirY*230, life:1.1}`.
  On impact: `dmg = max(1, pr.dmg - monster.armor)`, cyan damage number.
- **Melee:** `wp = weapon().wp + floor(str/4) + (cross?3:0)`. For each monster
  within distance `< 24` **and** `inFront(m)`: `dmg = max(1, wp - monster.armor)`;
  `m.hp -= dmg`, `m.hitT=0.15`, **yellow** damage number; if `m.hp<=0` →
  `killMonster(m)`. If anything was hit, set `flash=1, flashT=0.18`.
  *(v1.4: replaced the old crit system with the authentic armor-reduction
  formula. Melee reach is `<24`.)*

`inFront(mx,my)` — the monster must be in the facing direction:
```
facing left:  dx<-4 && |dy|<15
facing right: dx> 4 && |dy|<15
facing up:    dy<-4 && |dx|<15
facing down:  dy> 4 && |dx|<15
```

### 8.3 Monster attacks (`monsterAttack(m)`) — v1.4
- **Hit chance:** roll `1..31`; if `roll <= agility` the player **dodges** (no
  flash, no damage). Otherwise the hit lands.
- **Damage:** `dmg = monster.wp - ARMORS[p.armor].ap - (shield?SHIELDS[shield].sp:0)`;
  if a second `1..31` roll `< luck`, subtract `level` (lucky bonus).
  `damagePlayer(max(0, dmg))`.
- Traps use the same damage path via `monsterAttackDmg(baseDmg)` (no hit-chance
  roll — traps always apply damage).

### 8.4 Taking damage (`damagePlayer(n)`)
- If `invuln>0`, ignore (i-frames).
- `hp -= n`; `invuln=0.6`; `flash=2, flashT=0.22`; spawn a **red** damage
  number on the player (only if `n>0`).
- If `hp<=0`: `lives--`. If `lives<=0` → `state='over'`. Else respawn:
  `hp=maxHp()`, teleport to room 0 center, `invuln=2`, message `You were slain!`.

---

## 9. Combat Feedback (damage numbers & flashes)

**Floating damage numbers** (`dmgNums`):
- `addDmgNum(x, y, amt, color)` pushes `{x, y, amt, color, life:0.7}`.
- `updateDmgNums(dt)`: each number `life -= dt`, `y -= 24*dt` (rises), removed
  when `life<=0`.
- **Colors:** yellow = melee hit on monster; cyan = bow hit on monster; red =
  damage to the player.
- **Render:** alpha = `clamp(life/0.45, 0, 1)`; drawn with `textOutlined`
  (black outline) centered on the number's screen position.

**Screen flash** (drawn last in `renderPlay`):
- `flash===1` → `rgba(60,200,80,0.25)` (green, you landed a hit).
- `flash===2` → `rgba(200,50,50,0.30)` (red, you were hit).
- Only drawn while `flashT>0`.

**Monster hit flash:** while `m.hitT>0`, the monster is drawn at 50% alpha and
a white "spark" (a 3×3 center pixel plus four 1×1 pixels N/S/E/W) is drawn on
top.

**Scoring:** `killMonster(m)` removes the monster and adds **`+5`** to score
(monsters give no material value, matching the genre).

---

## 10. Monsters (AI)

`updateMonsters(dt)` — for each monster:
1. Decrement `hitT`, `attackCd`.
2. `d = dist(monster, player)`.
3. **Aggro:** if `d < m.aggro` **and** `lineOfSight(monster→player)` →
   `aggroed=true`. If `aggroed && d > m.lose` → `aggroed=false`.
4. **Velocity:**
   - If aggroed and `d>1`: move straight toward the player (normalized).
   - Else (wandering): every `wanderT` (re-randomized to `1 + random*2`
     seconds) pick a random angle; move along that direction.
5. `moveEntity(m, vx*m.spd*dt, vy*m.spd*dt)`.
6. **Bite:** if `d < p.r + m.r + 1` and `attackCd<=0` → `attackCd=0.8`,
   `damagePlayer(m.dmg)`.

`lineOfSight(x0,y0,x1,y1)` — sample the segment in steps of ≤8px; if any
sampled tile is solid, no LOS.

---

## 11. Items, Doors, Traps, Secrets

### 11.1 Pickup (`updateItems` + `pickup`)
Each item within `p.r + 6` of the player is picked up and removed from the map
(always spliced out, even if the player already had it):
- `treasure` → `score += value*level`; message `+V  <Material> <Kind>`.
- `dagger`/`shortsword`/`longsword`/`twohand`/`bow` → if not already owned,
  append to `weapons` and select it; message `Got <Name>`. If already owned,
  message `Already have <Name>` (item still removed from map).
- `arrows` → `arrows += n`; message `+n arrows`.
- `magicarrows` → `magicArrows += n`; message.
- `potion` → `potions++`; message `Potion`.
- `key` → `keys++`; message `Key`.
- `leather`/`chain`/`breast`/`plate` → set `p.armor`; message.
- `small`/`large` → set `p.shield`; message.
- `cross` → `p.cross=true`; message.
- `strength`/`agility`/`luck` → increment that stat by 2; message.
- `wand` → `p.wand++`; message.
- `stun`/`confuse`/`map`/`protect`/`teleport`/`blast` → increment
  `p.spells[type]` count; message `<Name> found!`.

### 11.2 Doors
Closed `DOOR` tiles block movement (solid). Opened by standing adjacent and
facing them with a key (§8.1). Rendered as a brown plank with a yellow knob.

### 11.3 Traps
Level-gated traps (pit/stone/freeze/death), `level+2` per dungeon (max 16), each
assigned a type from `TRAPS` of level ≤ current. Triggered once when stepped on
(§8.1) **unless `p.protectT>0`** (Protect spell). Damage traps use the
monster-attack formula (§8.3); freeze sets `p.freezeT`; death is instant.
**Locate Traps** (menu) reveals all traps in the dungeon at once.

### 11.4 Secrets
`SECRET` tiles look like normal floor. **Search Secret** (menu) scans a 7×7
area around the player; the first `SECRET` found becomes `FLOOR` and spawns a
new random item there; message `Secret found!`.

### 11.5 Spells (v1.5)
Six spell types (`SPELLS`). Finding a spell item increments `p.spells[type]`
(a count object, e.g. `{stun:2, map:1}`). Casting from the Spells menu
consumes one charge (`p.spells[type]--`). Durations use the original's unit
`U = 0.5s`:
```
stun:     freeze all monsters for (luck+agi+wand)/2 * U; also sets p.stunT
confuse:  visible monsters (lineOfSight) get confuseT=9999 (wander forever)
map:      set revealed[] = 1 for every tile (whole dungeon shown)
protect:  p.protectT = (luck+wand)*10 * U (traps disabled while >0)
teleport: move player to a random FLOOR tile >=25% of map size from the border
blast:    set 3 tiles in the facing direction to FLOOR (tunnel a corridor)
```
- Spell-effect timers `p.stunT`, `p.confuseT`, `p.protectT` decrement in
  `updatePlayer` and **carry over** between dungeons.
- A **Spells** submenu (`M` → Spells, `Esc` or `M` to close) lists only the
  spells the player has collected, with counts (e.g. "Stun Spell x2"). Selecting
  one casts it and consumes a charge. `spellMenu`/`spellIdx` are the submenu
  state; `renderMenu` draws whichever list is active. If no spells are held,
  the menu shows "No spells found".

### 11.6 Boss (v1.5)
Each level's final dungeon (`dungeon===16`) spawns a boss from `BOSSES[level]`
(Giant, Ogre, Giant, Priest, Warrior, Vampire, Warlord, Dragon) near the exit.
The boss (`type:'boss'`, `boss:true`) has its own `hp`/`wp`/`armor`/`r`, a large
`aggro` (200) and `lose` (9999) so it **always chases**, and a distinct
golden-crowned sprite. While a boss is alive, `renderHUD` draws a boss health bar
(name + red bar) in the bottom-left. `monsterAttack` uses the monster's own
`wp`/`armor` so bosses hit harder than regular monsters.

---

## 12. Progression & Timer

- **Timer:** `timeLeft` starts at `390` (6:30) per dungeon. Each frame (in
  play) it decreases by `dt`. When it hits 0 → `advanceLevel('time')`.
- **Structure:** 8 levels × 16 dungeons. `advanceLevel(how)`: snapshots the
  current inventory into `window.__carry` (v1.3) before calling `newLevel`, so
  the loadout carries over.
  ```
  dn = dungeon+1; lv = level
  if dn>16: dn=1; lv=min(lv+1, 8)
  newLevel(lv, dn)
  ```
  Message depends on `how`: `exit` → `You found the gate!`, else
  `On to the next dungeon...`.
- **Ways to advance:** reach the `EXIT` tile, the timer runs out, or the
  **Next Level** menu action.
- **Score** carries across dungeons; it is the "Gold" shown in the HUD and on
  the game-over screen.

---

## 13. Actions & Menu

The action menu (`M`) lists, in order:
```
Status          → showStatus()
Drink Potion    → usePotion()
Cycle Weapon    → cycleWeapon()
Locate Traps    → locateTraps()
Search Secret   → searchSecret()
Drop Item       → dropItem()
Next Level      → advanceLevel('menu')
Close           → menu=false
```

Behaviors:
- **showStatus:** opens a panel with 10 lines: Strength, Agility, Luck,
  Health `hp/max`, Lives, Weapon, Arrows, Keys, Potions, Gold.
- **usePotion:** if `potions>0` and `hp<max` → `potions--`, `hp=min(max, hp+4)`,
  message `Potion: +4 HP`; else `No potions`.
- **cycleWeapon:** if only one weapon → `Only fists`; else advance `weaponIdx`
  (wrap), message `Weapon: <Name>`.
- **locateTraps:** reveal every `TRAP` tile; message `Found N trap(s)` or
  `No traps here`.
- **searchSecret:** see §11.4.
- **dropItem:** if more than one weapon, remove the current one from `weapons`
  (adjust index) and drop it at the player's position; else `Nothing to drop`.
- **Next Level:** `advanceLevel('menu')`.

**Shortcuts (no menu):** `P` = use potion, `F` = cycle weapon.

---

## 14. Input Handling

- Global `keydown`/`keyup` maintain `keys[code]` (held) and `just[code]`
  (pressed this frame; cleared each frame by `clearJust()`).
- `Space` and the four arrows call `preventDefault()` (to stop page scroll).
- On `keydown`, `onKey(code)` routes by state:
  - **title:** Up/W = `titleDungeon=(titleDungeon+15)%16`; Down/S =
    `+1 %16`; Enter/Space = `startGame()`.
  - **over:** Enter/Space → `state='title'`.
  - **panel open:** M/Enter/Space/Esc → close panel.
  - **play:** Esc → toggle `paused`. If paused, ignore the rest. `M` → toggle
    menu (reset index). If menu open: Up/W and Down/S move selection,
    Enter/Space runs the selected action. Else `P` = potion, `F` = weapon.

---

## 15. Rendering Pipeline

`render()` clears to black, then:
- If `state==='title'` → `renderTitle()` and return.
- Else `renderPlay()`; if `state==='over'` overlay `renderGameOver()`; then draw
  `panel` (if any) else `menu` (if open) else `PAUSED` (if paused).

### 15.1 `renderPlay()`
Order (later = on top):
1. `renderMap(cx,cy)` — visible tiles only.
2. Items (`drawItem`).
3. Projectiles — 3×3 yellow squares.
4. Monsters (`drawMonster`), facing derived from `p.x < m.x ? 'left':'right'`.
5. Hero (`drawHero`).
6. Damage numbers (`dmgNums`, §9).
7. `renderHUD()`.
8. `renderMinimap()` (§15.11) — corner map, skipped when `minimap` is false.
9. `renderMsgs()`.
10. Screen flash overlay (§9).

### 15.2 `drawTile(tx,ty,sx,sy)` (per-tile art)
- **WALL:** base `[50,50,64]`; a 1px top-edge highlight `[80,80,100]` then
  `[66,66,84]`; bottom 2px `[33,33,45]` (shadow); a 1px vertical seam at `sx+7`.
  (Brick-like look. v1.2: added the top-edge highlight for depth.)
- **FLOOR:** base `[24,24,34]`; a subtle checkerboard — tiles where `(tx+ty)&1`
  are `[29,29,42]`; inner `[31,31,45]` (1px inset); plus a sparse deterministic
  speckle (1px `[38,38,54]` at a hash-derived offset) for texture. *(v1.2: floor
  no longer reads as flat void.)*
- **DOOR:** framed doorway — dark opening `(TILE-4)²` at `+2,+2`; wood frame
  (lintel across the top, left/right jambs); two vertical planks; a yellow
  keyhole. *(v1.1: redrawn to read as a door, see §21.1)*
- **TRAP:** if revealed → red `(TILE-6)²` at `+3,+3`; else a subtle 4×4 dark
  `[26,26,38]` at `+6,+6` (nearly invisible).
- **EXIT:** cyan `(TILE-4)²` at `+2,+2`; a pulsing white inner square
  (alpha `0.3 + 0.5*(0.5+0.5*sin(tGlobal*4))`).

### 15.3 `drawItem(it,sx,sy)` (per-type pixels)
Every item first draws a **faint 10×10 backing plate** (`[12,12,20]`) behind the
icon so small art reads against the floor. *(v1.2)* Then, per type:
- `treasure`: chest — brown body + lid, material-color band, yellow clasp.
- `potion`: flask — lgray cork, red neck + bulb, lred shine.
- `key`: yellow bow (with hole) + shaft + teeth.
- `arrows`: brown quiver + two lgray shafts + red fletch.
- `shortsword`/`longsword`: lgray blade + white edge + brown guard + pommel.
- `bow`: brown curved stave + lgray string + nocked arrow (shaft + white head).
- `strength`: red diamond; `agility`: green chevron; `luck`: four-leaf clover.
  *(v1.1: every icon is distinct, see §21.1)*

### 15.4 Hero & monster sprites
- `drawHero(x,y)`: while `invuln>0`, blink (skip drawing every other 1/20s).
  Draw the hero sprite (facing-aware) + a 3×3 lgray "sword" offset in the facing
  direction.
- `drawMonster(m,x,y,facing)`: §9 hit-flash handling + sprite.

*(v1.3: the 1px `spriteOutline` ring added in v1.2 was removed — it looked like a
box around the sprites.)*

### 15.5 HUD (`renderHUD`)
Top 12px black bar + 1px dgray line at y=12. Text baseline `y=2`, left-to-right
at these x offsets:
```
x=4    L<level> D<dungeon 2-digit>      white
x=42   HP <hp>/<max>                    green, or lred if hp<max
x=94   LIVES <n>                         white
x=140  GOLD <score>                      yellow
x=204  <mm:ss>                          cyan, or red if <30s
x=232  <WEAPON NAME, uppercase>          lgray
x=266  x<arrows>                         white (only if arrows>0)
```
`fmtTime(s)` → `ceil`, `m:ss` with zero-padded seconds.

### 15.6 Messages (`renderMsgs`)
Bottom-left stack. Start `y = H-10 - len*9`; each message: a
`rgba(0,0,0,0.6)` background box, yellow text at `x=7`, advance `y` by 9.

### 15.7 Menu (`renderMenu`)
Centered box `bw=124`, `bh=MENU.length*10+16`. Cyan border + title
`MENU  (M to close)`. Each item 10px apart; the selected one is yellow with a
`> ` prefix, others white with `  `.

### 15.8 Panel (`renderPanel`)
Centered box `bw=140`, `bh=lines*10+20`. Yellow border + title; white lines;
`(M to close)` in lgray at the bottom.

### 15.9 Title (`renderTitle`)
```
y=28  'G A T E W A Y'            cyan, scale 3
y=66  'an original dungeon crawler'   lgray
y=96  'Dungeon: NN'              yellow
y=118 '[Up/Down] choose    [Enter] enter'  white
y=150 'Find treasure. Fight monsters. Beat the clock.'  dgray
y=162 'An HTML5 tribute - not the 1983 game.'  dgray
```

### 15.11 Minimap (`renderMinimap`) — v1.1, fog-of-war v1.5

A live overview of the dungeon, drawn in the **bottom-right** corner
(under the HUD) so it never collides with the centered menu/panel.

- **Offscreen canvas** `mmcv` (`MAP_W × MAP_H`, one pixel per tile) is built
  each frame with `createImageData`/`putImageData`, then blitted scaled 2×
  (`s=2`) so the map is `MAP_W*2 × MAP_H*2` = `96 × 68` px. Origin
  `ox = W - mw - 4`, `oy = 16`.
- **Fog of war:** only tiles with `revealed[]===1` are drawn; unexplored tiles
  are black `[8,8,12]`. Tiles are revealed by `revealAround(tx,ty,3)` in
  `updatePlayer` (3-tile radius as the player walks) and by the **Map** spell
  (reveals the entire dungeon).
- **Tile colors** (1 px each, revealed only): wall `[74,74,92]`,
  floor/secret `[30,30,42]`, door `[150,100,50]`, exit `[0,220,220]`,
  trap = red `[220,40,40]`.
- **Entities** (drawn over the map, scaled by `s`, revealed tiles only):
  monsters = 2×2 red; items = 1×1 yellow; player = 3×3 white with a 1px
  black outline (always shown).
- **Frame:** a semi-transparent black backing (`rgba(0,0,0,0.66)`) with a
  light-grey border, and a `MAP (K)` label above it.
- Toggled by **K** (§18); hidden when `minimap` is false.

### 15.10 Game Over (`renderGameOver`)
```
overlay rgba(0,0,0,0.72)
y=66  'GAME OVER'   red, scale 3
y=100 'Gold: N'     yellow
y=122 '[Enter] title'  white
```

---

## 16. Update Pipeline

`update(dt)`:
```
tGlobal += dt
flashT -= dt (if >0)
age & filter msgQ
if state!=='play' or paused or panel or menu: return
timeLeft -= dt; if <=0: advanceLevel('time'); return
updatePlayer(dt)          # may advanceLevel / change state
if state!=='play': return
updateMonsters(dt)
updateProjectiles(dt)
updateDmgNums(dt)
updateItems()
```

`updateProjectiles(dt)`: move each, age it; it dies on `life<=0`, on hitting a
solid tile, or on hitting a monster (dist `< m.r+3`): apply `dmg`, set
`hitT=0.15`, spawn a **cyan** damage number, kill if `hp<=0`.

---

## 17. Demo / Preview Modes (optional, URL query)

An IIFE at the end of the script reads `location.search`:
- **`?demo=1`** — auto-plays: `startGame()`, then every 240ms pick a random
  direction (W/D/S/A) and press it, and press Space ~50% of the time. Used for
  attract mode and headless screenshots.
- **`?hit=1`** — combat preview: `startGame()`, give the player a short sword,
  then every 120ms keep a goblin 14px in front of the hero (facing right) and
  press Space. Produces a continuous stream of damage numbers for screenshots.
- No query param → normal game (title screen).

These are harmless and only activate with the query string.

---

## 18. Controls Reference

| Action | Keys |
|---|---|
| Move | Arrows or WASD |
| Attack / shoot | Space or X |
| Action menu | M |
| Use potion | P |
| Cycle weapon | F |
| Toggle minimap | K |
| Toggle sound | Z |
| Pause / resume | Esc |
| Title: choose dungeon | Up / Down |
| Title / Game-over: confirm | Enter or Space |

---

## 19. Verification & Testing

Because the game is pure JS with a tiny DOM surface, it can be exercised
headlessly without a browser:

> All scratch/verification files live in the project's **`_tmp/`** folder
> (not `/tmp`). Keep temporary files there; they are not part of the game.

1. **Syntax check.** Extract the `<script>` body to `_tmp/apshai_check.js` and
   run `node --check _tmp/apshai_check.js`.
2. **Logic harness.** Stub `document`, `canvas.getContext` (no-op 2D context),
   `window`, `performance`, and `requestAnimationFrame` (see `_tmp/stub.js`),
   then load the script in Node and drive it: `startGame()`, run `update(dt)` for N frames, call
   actions (`showStatus`, `usePotion`, `cycleWeapon`, `locateTraps`,
   `searchSecret`, `dropItem`, `advanceLevel`), and assert invariants
   (state, hp, score, monster/item/room counts, damage numbers created on hit,
   damage numbers expiring).
3. **Visual check.** Headless Chromium screenshot:
   ```
   chromium --headless --no-sandbox --disable-gpu --hide-scrollbars \
     --window-size=960,640 --virtual-time-budget=3000 \
     --screenshot=out.png "file://.../apshai.html"            # title
   ... "file://.../apshai.html?hit=1"                          # combat
   ```
   Confirm the title, HUD, map, sprites, and damage numbers render crisply.

A passing harness should report, for example: `startGame OK`, a few seconds of
`update` keeps `state='play'`, `showStatus` yields 10 lines, a forced melee hit
creates a yellow damage number and reduces monster hp, and numbers expire after
~1s.

---

## 20. Folder Layout

| File | Purpose |
|---|---|
| `apshai.html` | The game (this spec). |
| `apshai (copy).html` | Backup of the v1 build (before the icon + minimap pass). |
| `PLAN.md` | This document. |
| `_tmp/` | **Temporary** working folder (JS extracts, Node stub harness, screenshot crops). Not part of the game — safe to delete. |

Verification screenshots are taken on demand into `_tmp/` (see §19) and are
not kept in the folder.

---

## 21. Suggested Future Extensions (not in v1)

### 21.1 Icon / sprite improvements — DONE in v1.1

The icons that read poorly at small size were redrawn so each object is
recognizable at a glance. Current implementations:

- **Door** (§15.2): a framed doorway — dark opening, wood frame (lintel + two
  jambs), two vertical planks, and a yellow keyhole. Distinct silhouette that
  reads even when the player is not next to it.
- **Bow** (§15.3): a curved brown stave (right arc) with a vertical lgray
  string and a nocked arrow (shaft + white head) — clearly different from the
  straight sword.
- **Pickup icons** (§15.3), each unique: treasure = chest (brown body + lid +
  material band + clasp); potion = flask (cork + neck + bulb + shine); key =
  bow + hole + shaft + teeth; arrows = quiver + two shafts + red fletch;
  sword = blade + edge + guard + pommel; strength = red diamond; agility =
  green chevron; luck = four-leaf clover.

**Acceptance test (met):** from a fresh screenshot, a viewer can name the door,
the bow, and every pickup icon without being shown a legend.

### 21.2 Other extensions

- *(Done in v1.2: crits — luck-driven 2× hits with a bigger white number.)*
- *(Done in v1.4: 15 monster types.)*
- *(Done in v1.5: boss per level's final dungeon.)*
- *(Done in v1.6: audio — see §22.)*
- *(Done in v1.5: walk-bob animation on hero and monsters.)*
- **Persistence:** high score via `localStorage`.
- **Touch controls** for mobile.
- **Remaining balance/look polish:** deeper difficulty tuning across all 8
  levels; more monster AI variety (attack patterns, ranged monsters).

---

## 22. Audio (v1.6 — procedural WebAudio chiptune)

All audio is **synthesized at runtime** with the WebAudio API — no audio files,
no libraries, keeping the single-file constraint. Nothing is copied from the
original game.

### 22.1 Context & master chain
- `audioInit()` creates the `AudioContext` **lazily on the first keydown**
  (browsers block audio until a user gesture) and resumes it if `suspended`.
  In headless/demo mode (no gesture) `AC` stays null and every audio call is a
  no-op — the game runs silently, which is why the Node harness and
  `?demo=1` screenshots work unchanged.
- One master `GainNode` at **0.15** → destination. All SFX and music route
  through it. Mute (**Z**) sets the master gain to 0/0.15 and flips `muted`
  (SFX and music scheduling also check `muted` to avoid wasted nodes).

### 22.2 SFX (`SFX` object — 15 one-shots)
Each is a few lines of oscillator/noise + gain envelope (fast attack,
exponential decay to 0.001 → no clicks):

| SFX | Recipe |
|---|---|
| `swing` | 80 ms highpassed noise burst (whoosh) |
| `shoot` | square 700→300 Hz, 80 ms |
| `hit` | square 400 Hz, 60 ms |
| `kill` | square 500→200 Hz, 120 ms |
| `hurt` | sawtooth 200→50 Hz, 250 ms |
| `die` | sawtooth 300→40 Hz, 600 ms + noise |
| `pickup` | square C5→G5 two-note, 60 ms apart |
| `treasure` | square C5–E5–G5 arpeggio |
| `door` | sine 120→80 Hz, 150 ms (thunk) |
| `trap` | square 160→80 Hz, 300 ms (buzz) |
| `spell` | sine 400→1000 Hz rising sweep, 300 ms |
| `select` | square 880 Hz, 40 ms (menu confirm) |
| `move` | square 440 Hz, 30 ms (menu cursor) |
| `levelup` | C5–E5–G5–C6 arpeggio, 90 ms apart |
| `boss` | sawtooth A2–A2–C3–E3 fanfare, 120 ms apart |

Hook points: swing/shoot/hit in `tryAttack`; kill in `killMonster`; hurt/die in
`damagePlayer`; treasure/pickup in `pickup`; door in `updatePlayer`; trap in
`triggerTrap`; spell in `castSpell`; levelup in `advanceLevel`; boss in
`placeEntities`; select/move throughout `onKey` (menu + title navigation).

### 22.3 Music (step sequencer)
- **Patterns:** `MUSIC.dungeon` and `MUSIC.boss` — each an 8-bar loop of 64
  eighth-note steps, two voices: `bass[]` (triangle) and `mel[]` (square;
  sawtooth in boss mode). `0` = rest; values are MIDI note numbers rendered by
  `mf(m) = 440·2^((m−69)/12)`.
- **Dungeon theme:** A minor, 110 BPM, sparse melody over a root-note bass.
- **Boss theme:** same melody, 132 BPM, denser 16th-feel bass, sawtooth lead.
- **Scheduler** ("A Tale of Two Clocks" pattern): `setInterval` every 25 ms
  calls `musicTick()`, which schedules notes while `musicNextT <
  AC.currentTime + 0.12` (120 ms lookahead) using absolute `ctx.currentTime`
  timestamps — sample-accurate regardless of timer jitter. If the tab is
  throttled and `musicNextT` falls >200 ms behind, it resyncs to now instead of
  dumping a burst of late notes.
- **Lifecycle:** `startGame()` → `startMusic('dungeon')`; boss spawn →
  `startMusic('boss')`; boss killed or dungeon advanced (non-boss) → back to
  `'dungeon'`; game over → `stopMusic()`. Music keeps playing while paused.

### 22.4 Mute & UI
- **Z** toggles `muted` in any state; shows a message and, while muted, a red
  `MUTED (Z)` indicator bottom-right of the play area (clear of the minimap).
- The HTML hint bar lists `Sound Z`.

---

## Appendix A — Full 5×7 FONT table

Each glyph is 7 rows of 5 characters (`1` = pixel on). Copy verbatim.

```js
const FONT = {
' ': ["00000","00000","00000","00000","00000","00000","00000"],
'!':["00100","00100","00100","00100","00100","00000","00100"],
'"':["01010","01010","00000","00000","00000","00000","00000"],
"'" :["00100","00100","00000","00000","00000","00000","00000"],
'(' :["00010","00100","01000","01000","01000","00100","00010"],
')' :["01000","00100","00010","00010","00010","00100","01000"],
'*':["00000","10101","01110","11111","01110","10101","00000"],
'+':["00000","00100","00100","11111","00100","00100","00000"],
',':["00000","00000","00000","00000","01100","00100","01000"],
'-':["00000","00000","00000","11111","00000","00000","00000"],
'.':["00000","00000","00000","00000","00000","01100","01100"],
'/':["00001","00010","00010","00100","01000","01000","10000"],
':':["00000","01100","01100","00000","01100","01100","00000"],
';':["00000","01100","01100","00000","01100","00100","10000"],
'?':["01110","10001","00001","00110","00100","00000","00100"],
'[':["00110","00100","00100","00100","00100","00100","00110"],
']':["01100","00010","00010","00010","00010","00010","01100"],
'0':["01110","10001","10011","10101","11001","10001","01110"],
'1':["00100","01100","00100","00100","00100","00100","01110"],
'2':["01110","10001","00001","00110","01000","10000","11111"],
'3':["11111","00010","00100","00010","00001","10001","01110"],
'4':["00010","00110","01010","10010","11111","00010","00010"],
'5':["11111","10000","11110","00001","00001","10001","01110"],
'6':["00110","01000","10000","11110","10001","10001","01110"],
'7':["11111","00001","00010","00100","01000","01000","01000"],
'8':["01110","10001","10001","01110","10001","10001","01110"],
'9':["01110","10001","10001","01111","00001","00010","01100"],
'A':["01110","10001","10001","11111","10001","10001","10001"],
'B':["11110","10001","10001","11110","10001","10001","11110"],
'C':["01110","10001","10000","10000","10000","10001","01110"],
'D':["11100","10010","10001","10001","10001","10010","11100"],
'E':["11111","10000","10000","11110","10000","10000","11111"],
'F':["11111","10000","10000","11110","10000","10000","10000"],
'G':["01110","10001","10000","10111","10001","10001","01111"],
'H':["10001","10001","10001","11111","10001","10001","10001"],
'I':["01110","00100","00100","00100","00100","00100","01110"],
'J':["00111","00010","00010","00010","00010","10010","01100"],
'K':["10001","10010","10100","11000","10100","10010","10001"],
'L':["10000","10000","10000","10000","10000","10000","11111"],
'M':["10001","11011","10101","10101","10001","10001","10001"],
'N':["10001","10001","11001","10101","10011","10001","10001"],
'O':["01110","10001","10001","10001","10001","10001","01110"],
'P':["11110","10001","10001","11110","10000","10000","10000"],
'Q':["01110","10001","10001","10001","10101","10010","01101"],
'R':["11110","10001","10001","11110","10100","10010","10001"],
'S':["01111","10000","10000","01110","00001","00001","11110"],
'T':["11111","00100","00100","00100","00100","00100","00100"],
'U':["10001","10001","10001","10001","10001","10001","01110"],
'V':["10001","10001","10001","10001","10001","01010","00100"],
'W':["10001","10001","10001","10101","10101","11011","10001"],
'X':["10001","10001","01010","00100","01010","10001","10001"],
'Y':["10001","10001","01010","00100","00100","00100","00100"],
'Z':["11111","00001","00010","00100","01000","10000","11111"],
'a':["00000","00000","01110","00001","01111","10001","01111"],
'b':["10000","10000","10110","11001","10001","10001","11110"],
'c':["00000","00000","01110","10000","10000","10001","01110"],
'd':["00001","00001","01101","10011","10001","10001","01111"],
'e':["00000","00000","01110","10001","11111","10000","01110"],
'f':["00110","01001","01000","11100","01000","01000","01000"],
'g':["00000","01111","10001","10001","01111","00001","01110"],
'h':["10000","10000","10110","11001","10001","10001","10001"],
'i':["00100","00000","01100","00100","00100","00100","01110"],
'j':["00010","00000","00110","00010","00010","10010","01100"],
'k':["10000","10000","10010","10100","11000","10100","10010"],
'l':["01100","00100","00100","00100","00100","00100","01110"],
'm':["00000","00000","11010","10101","10101","10001","10001"],
'n':["00000","00000","10110","11001","10001","10001","10001"],
'o':["00000","00000","01110","10001","10001","10001","01110"],
'p':["00000","00000","11110","10001","11110","10000","10000"],
'q':["00000","00000","01101","10011","01101","00001","00001"],
'r':["00000","00000","10110","11001","10000","10000","10000"],
's':["00000","00000","01111","10000","01110","00001","11110"],
't':["01000","01000","11100","01000","01001","01001","00110"],
'u':["00000","00000","10001","10001","10001","10011","01101"],
'v':["00000","00000","10001","10001","10001","01010","00100"],
'w':["00000","00000","10001","10001","10101","10101","01010"],
'x':["00000","00000","10001","01010","00100","01010","10001"],
'y':["00000","00000","10001","10001","01111","00001","01110"],
'z':["00000","00000","11111","00010","00100","01000","11111"],
};
```

---

## Appendix B — Sprite pixel maps

Character meanings are in the LEGEND table (§4.3). `.` = transparent. Copy
verbatim. `makeSprite(rows)` renders each row's characters to 1px cells.

```js
const HERO = [
"....ssss....","...ssssss...","...soooos...","...soooos...",
"....oooo....","..bbbbbbbb..",".bbbbbbbbbb.",".bbbbbbbbbb.",
"..bbbbbbbb..","..nnnnnnnn..","...nn..nn...","...nn..nn...",
"..nnn..nnn..","..nnn..nnn.."
];
const GOBLIN = [
".gg......gg.",".ggg....ggg.","..gggggggg..","..gkggggkg..",
"..gggggggg..","...grrrrg...","..gggggggg..",".gggggggggg.",
".gg..gg..gg.",".gg..gg..gg.","gg....gg....","............"
];
const ORC = [
"..nnnnnnnnnn..",".nnnnnnnnnnnn.",".nnknnnnnnknn.",".nnnnnnnnnnnn.",
".nnnnrrrrnnnn.","..nnnnnnnnnn..",".rrrrrrrrrrrr.","rrrrrrrrrrrrrr",
"rrrrrrrrrrrrrr",".rrrrrrrrrrrr.",".nnn....nnn...",".nnn....nnn...",
"nnnn....nnnn..","nnnn....nnnn.."
];
```

---

*End of specification.*
