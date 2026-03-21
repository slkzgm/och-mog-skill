# Strategy And Gotchas

Use this file for backend quirks, retry behavior, and concise autoplay defaults. For the fuller strategic model, also read `strategy-core.md` and `upgrade-mapping.md`.

## Recommended Default Policy

Unless the user requests a different objective, optimize for `treasure per key` with robust play.

Default decision order:

1. resolve pending upgrade selection
2. follow the floor-phase policy from `strategy-core.md`
3. if stairs are known, path toward stairs first unless stronger combat EV or sustain logic overrides it
4. attack adjacent high-value or dangerous follower enemies
5. if stairs are unknown, expand the frontier to reveal more map
6. break adjacent `pot` or `crate`
7. move toward useful non-breakable interactives such as fountains when sustain matters
8. `pass` only as a strategic last resort or a deliberate combat-micro choice

## Upgrade Priority

Current priority list:

- `treasure_hunter`
- `lucky_find`
- `scavenger`
- `magnet`
- `quick_step`
- `second_wind`
- `portal_adept`
- `attack_up`
- `tough_hide`
- `trap_sight`
- `soft_traps`
- `retaliation`

If none of the preferred upgrades appear, select the best remaining option rather than stalling.

## Reroll Rules

Observed reroll cost sequence:

- `10`
- `20`
- `30`
- `50`
- `80`
- then Fibonacci-style continuation

Recommended rule:

- reroll only when upgrade ROI is meaningfully better and the treasure cost is affordable
- do not reroll blindly just because the option exists

## High-Value Enemy Hints

Enemy labels that have been treated as higher value in our automation:

- `nian`
- `lucky`
- `fleeing`
- `maomi`
- `treasure`

Treat these as hints, not guaranteed payout rules.

## Critical Backend Quirks

Observed in practice:

- `GET /keys/balance` can intermittently return `500`
- `POST /runs/create` can intermittently return `500`
- `POST /runs/:runId/move` can intermittently return `500`
- bursty actions can produce `409 LOCK_CONFLICT`
- invalid or replayed `upgrade_selected` inputs have triggered `500 INTERNAL_ERROR`

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
5. avoid repeating the same failed move target immediately unless the state changed

## Why Auto-Pass Is Wrong

Auto-`pass` after a backend error is actively harmful:

- it burns energy
- enemies may act
- treasure-per-key drops
- the failed action may have been legal, but the backend was temporarily unstable

## Session Discipline

- Keep one stable session cookie for the whole run batch.
- Re-auth only when necessary.
- Do not interleave multiple simultaneous action requests for the same run.

## Logging Guidance

If the agent can persist notes, log at least:

- run id
- floor
- turn
- player position
- energy
- treasure
- chosen action
- target metadata
- response events
- resource deltas
- backend error details

This is especially useful when refining autoplay policy.
