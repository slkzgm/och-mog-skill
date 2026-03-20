# Game State And Map

Use this file when the agent needs to interpret `gameState`, render the map mentally, or decide which action type is legal.

## Core State Fields

Read these on every turn:

- `gameState.runId`
- `gameState.status`
- `gameState.currentFloor`
- `gameState.turnNumber`
- `gameState.keysUsed`
- `gameState.player`
- `gameState.pendingUpgradeOptions`
- `gameState.currentRerollCount`
- `gameState.mapData`
- `gameState.fogMask`
- `gameState.tileData`
- `gameState.enemies`
- `gameState.interactive`
- `gameState.pickups`
- `gameState.traps`
- `gameState.arrowTraps`
- `gameState.portals`
- `gameState.torches`

## Player Fields That Matter

From `gameState.player`:

- `x`, `y`
- `energy`, `maxEnergy`
- `treasure`, `marbles`, `hongbao`
- `attackPower`, `baseAttackPower`
- `upgrades`

## Map Encoding

Observed tile meanings:

- `0` = corridor, passable
- `1` = room, passable
- `2` = wall, blocked

## Fog Encoding

Observed meanings:

- `0` = hidden
- `1` = explored but not currently visible
- `2` or higher = visible

## Practical Map Legend

When rendering or reasoning about map cells, this legend is consistent with the custom client:

- `@` player
- `>` stairs
- `F` fountain
- `C` crate
- `P` pot
- `D` door
- `$` pickup
- `^` trap
- `A` arrow trap
- `O` portal
- `T` torch
- `E` enemy
- `G` ghost-like enemy
- `?` hidden fog
- `:` corridor
- `.` room
- `#` wall

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

These are critical:

- `pot` and `crate` are breakables: use `break`
- `stairs` are movement-based: move onto the tile
- `fountain` is movement-based: move onto the tile
- non-breakable interactives should generally be entered with `move`
- pickups are movement-based, not `break`

## Enemy Rules

An enemy is not a safe attack target if it is ghost-like.

Treat it as ghost-like if either:

- `type` contains `ghost`
- `spriteType` contains `ghost`
- `maxHp <= 0`

## Upgrade Flow Rules

If `pendingUpgradeOptions` is non-empty:

- stop normal navigation and combat
- choose `upgrade_selected` or `reroll`
- do not send `move`, `attack`, `break`, or `pass`

## Minimal Decision Procedure

For one directional step:

1. compute the adjacent target cell
2. if an adjacent valid enemy is there, use `attack`
3. else if an adjacent `pot` or `crate` is there, use `break`
4. else if the tile is passable, use `move`
5. else the direction is blocked

This action-selection contract matches the custom client and the agent CLI logic that were previously validated against the live game.

