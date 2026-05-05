# Strategy And Gotchas

Use this file for backend quirks, retry behavior, and concise autoplay defaults. For the fuller strategic model, also read `strategy-core.md` and `upgrade-mapping.md`.

## Recommended Default Policy

Unless the user requests a different objective, optimize for `treasure per key` with robust play.

Default decision order:

1. resolve pending upgrade selection
2. prevent immediate death with movement, combat, or items
3. follow the floor-phase policy from `strategy-core.md`
4. if stairs are known, path toward stairs unless stronger combat EV or sustain logic overrides it
5. attack adjacent high-value or dangerous follower enemies
6. use high-impact items when they clearly improve survival or premium-target EV
7. if stairs are unknown, expand the frontier to reveal more map
8. break adjacent `pot` or `crate`
9. move toward useful non-breakable interactives such as fountains when sustain matters
10. `pass` only as a strategic last resort or a deliberate combat-micro choice

## Upgrade Priority

Good default priority:

- `treasure_hunter`
- `lucky_find`
- `scavenger`
- `magnet`
- `quick_step`
- `second_wind`
- `portal_adept`
- `fatal_edge`
- `attack_up`
- `tough_hide`
- `trap_sight`
- `soft_traps`
- `retaliation`

If none of the preferred upgrades appear, select the best remaining option rather than stalling.

## Reroll Rules

Reroll cost sequence:

- `10`
- `20`
- `30`
- `50`
- `80`
- `130`
- Fibonacci-style continuation

World runs, where enabled, use the World resource cost at one tenth of the listed cost, minimum `1`.

Recommended rule:

- reroll only when upgrade ROI is meaningfully better and the cost is affordable
- do not reroll blindly just because the option exists
- avoid rerolling heavily before floor `7` unless it materially stabilizes the run

## High-Value Enemy Hints

Enemy labels that can indicate higher value or special behavior:

- `kingslime`
- `skeletonking`
- `nian`
- `lucky`
- `fleeing`
- `maomi`
- `treasure`

Treat these as hints, not guaranteed payout rules. Prefer current `gameState` and events.

## Critical Backend Quirks

Observed in practice:

- `GET /keys/balance` can intermittently return `500`
- `POST /runs/create` can intermittently return `500`
- `POST /runs/:runId/move` can intermittently return `500`
- bursty actions can produce `409 LOCK_CONFLICT`
- invalid or replayed `upgrade_selected` inputs can produce backend errors
- stale `turnNumber` can cause expected-turn mismatches

## Retry Policy

Treat these as retryable:

- `409`
- `429`
- `500`
- `502`
- `503`
- `504`
- `LOCK_CONFLICT`
- `INTERNAL_ERROR`

Recommended handling:

1. do not auto-`pass`
2. wait briefly with exponential backoff
3. refetch `GET /runs/:runId`
4. recompute the best action from fresh state
5. avoid repeating the same failed target immediately unless the state changed

## Why Auto-Pass Is Wrong

Auto-`pass` after a backend error is harmful:

- it burns energy
- enemies may act
- treasure-per-key drops
- the failed action may have been legal, but the backend was temporarily unstable

## Session Discipline

- Keep one stable session cookie for the whole run batch.
- Re-auth only when necessary.
- Do not interleave multiple simultaneous action requests for the same run.
- Include `expectedTurnNumber` when available to detect stale actions cleanly.

## Logging Guidance

If the agent can persist notes, log at least:

- run id
- run type
- floor
- turn
- player position
- energy
- treasure
- inventory item
- chosen action
- target metadata
- response events
- resource deltas
- backend error details

This is useful when refining autoplay policy.
