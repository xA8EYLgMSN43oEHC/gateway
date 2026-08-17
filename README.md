# GATEWAY

An **original** single-file HTML5 dungeon crawler in the spirit of the 1983
C64 classic. No dependencies, no build step, no asset files — every sprite,
sound effect, and music track is generated in code.

## Play

Open `apshai.html` in any modern browser. That's it.

## Controls

| Key | Action |
|---|---|
| Arrows / WASD | Move |
| Space / X | Attack (melee or bow) |
| M | Menu (status, weapon, spells, locate traps, search secrets) |
| P | Drink potion |
| F | Cycle weapon |
| K | Toggle minimap |
| Z | Mute sound |
| Esc | Pause / close menu |

## Gameplay

- **8 levels × 16 dungeons.** Fight monsters, grab treasure, open doors,
  avoid traps, and escape through the gate before the 390-second timer runs out.
- **Bosses** guard the final dungeon of every level.
- **Spells** (stun, confuse, map, protect, blast, teleport) are single-use
  items you find in dungeons — cast them from the M menu.
- **Equipment** (weapons, armor, shields, cross, wand) and stats carry over
  between dungeons.
- **Victory** is stepping through the gate of dungeon 8-16.

Handy URL shortcuts (for testing): `?level=8&dungeon=16` jumps straight to
the final boss, `?demo=1` starts in demo mode.

## Notes

- This is an original tribute: mechanics are inspired by the classic, but all
  code, art, names, and music are new.
- `PLAN.md` is the full implementation spec (game design, systems, balance
  tables, changelog).
