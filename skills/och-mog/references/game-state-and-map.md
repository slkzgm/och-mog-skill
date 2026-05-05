# Game State And Map

Use this file when the agent needs to interpret `gameState`, render the map mentally, or decide which action type is legal.

## Core State Fields

Read these on every turn:

- `gameState.runId`
- `gameState.status`
- `gameState.runType`
- `gameState.currentFloor`
- `gameState.turnNumber`
- `gameState.keysUsed`
- `gameState.player`
- `gameState.pendingUpgradeOptions`
- `gameState.currentRerollCount`
- `gameState.teleportUseCount`
- `gameState.mapData`
- `gameState.fogMask`
- `gameState.tileData`
- `gameState.wallTileData`
- `gameState.decorationTiles`
- `gameState.enemies`
- `gameState.interactive`
- `gameState.pickups`
- `gameState.traps`
- `gameState.arrowTraps`
- `gameState.portals`
- `gameState.torches`
- `gameState.pendingBombs`

The public response intentionally omits hidden seed and full generation internals. Do not ask the skill to reconstruct private generation state.

## Turn Order

Core loop:

- the player acts first
- then enemies act

Why this matters:

- stepping into adjacency can be meaningfully different from waiting
- `pass` is a real tactical action, not just a noop
- threat evaluation must account for enemy movement and retaliation after the player's choice

## Player Fields That Matter

From `gameState.player`:

- `x`, `y`
- `energy`, `maxEnergy`
- `treasure`, `marbles`, `hongbao`
- `amber`, `raffleTickets`, `abs`, `caps`
- `attackPower`, `baseAttackPower`
- `upgrades`
- `items`

Inventory is constrained. If the player already has an item and a better one appears, consider using or discarding strategically rather than ignoring all future drops.

## Map Encoding

Current public state uses:

- `0` = walkable floor
- `1` = wall or blocked base tile
- `2` = unexplored or hidden placeholder

Do not use legacy tile encodings for current action legality.

## Fog Encoding

Current meanings:

- `0` = hidden
- `1` = explored but not currently visible
- `2` = visible

Only treat entities as actionable if they are present in current authoritative state and not merely inferred from old fog.

## Practical Map Legend

When rendering or reasoning about map cells:

- `@` player
- `>` stairs
- `F` fountain
- `C` crate
- `P` pot
- `H` chest
- `R` rock
- `$` pickup
- `^` trap
- `A` arrow trap
- `O` portal
- `T` torch
- `E` enemy
- `G` ghost-like enemy
- `?` hidden fog
- `.` walkable floor
- `#` wall or blocker

## Direction And Targeting

Directions use grid deltas:

- `up` => `(x, y - 1)`
- `down` => `(x, y + 1)`
- `left` => `(x - 1, y)`
- `right` => `(x + 1, y)`

For a directional action:

1. compute the adjacent target cell
2. inspect enemies on that cell
3. inspect interactives on that cell
4. inspect passability of the tile
5. choose exactly one of `attack`, `break`, or `move`

## Interaction Rules

Critical distinctions:

- `pot` and `crate` are breakables: use `break`
- `chest` blocks movement but auto-opens when the player moves adjacent
- `rock` blocks movement and is not a normal `break` target
- `stairs` are movement-based: move onto the tile
- `fountain` is movement-based: move onto the tile
- `portal` is movement-based first; use the teleport endpoint only after the state prompts for teleport
- pickups are movement-based, not `break`

Fountains are one-time sustain nodes and can restore up to `10` energy on first use.

## Item Rules

Known active or inventory item ids include:

- `gas_pedal`
- `single_shot`
- `pocket_portal`
- `last_light`
- `survival_instinct`
- `bomb`
- `chain_shot`
- `piercing_shot`
- `escape_rope`
- `nine_lives`

Targeted items use `targetX` and `targetY`. Untargeted items omit target coordinates.

Default policy:

- use a defensive item before lethal damage when the item clearly prevents the loss
- use targeted damage items on high-value or dangerous enemies when line/target rules allow
- discard only when the current item is lower value than future item access or blocks the intended plan

## Enemy Rules

An enemy is not a safe attack target if it is invincible or ghost-like.

Treat it as ghost-like or invalid for normal attacks if either:

- `type` contains `ghost`
- `spriteType` contains `ghost`
- `maxHp <= 0`

Important stacking rule:

- if a normal enemy and an invincible enemy occupy the same cell, do not attack that cell unless the live state explicitly exposes a valid target that the server accepts.

Other important identifiers:

- `bat`: erratic follower from floor 2+
- `skeleton`, `skeleton2`: chasers from floor 4+
- `skullbat`: chaser with heavy-hit behavior from floor 5+
- `shroom`: stationary ranged enemy from floor 7+
- `kingslime`: high-value wobble enemy from floor 8+
- `mimic`: dormant enemy hidden as a chest until revealed unless mimic-sense state is active
- `skeletonking`: Sir Jackalot, fleeing jackpot boss on its designated floor
- `pengu`: event enemy
- `capfull`: event enemy

Additional tactical note:

- do not assume it is always best to step into adjacency first
- some enemies punish entering adjacent range, so waiting can be better when their movement pattern will bring them to you
- do not overuse this rule; avoid pass loops against enemies that will not converge

## Upgrade Flow Rules

If `pendingUpgradeOptions` is non-empty:

- stop normal navigation and combat
- choose `upgrade_selected` or `reroll`
- do not send `move`, `attack`, `break`, `pass`, `use_item`, `discard_item`, or `teleport`

## Minimal Decision Procedure

For one directional step:

1. compute the adjacent target cell
2. if an adjacent valid enemy is there, use `attack`
3. else if an adjacent `pot` or `crate` is there, use `break`
4. else if the tile is passable and not occupied by a blocker, use `move`
5. else the direction is blocked

This action-selection contract should always be checked against the current server state rather than stale client memory.
