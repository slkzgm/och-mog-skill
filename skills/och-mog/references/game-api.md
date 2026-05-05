# Game API

Use this file for run lifecycle, endpoint payloads, action contracts, teleport, item actions, and response parsing.

## Confirmed REST Endpoints

- `GET /status`
- `GET /runs/active`
- `GET /runs/active?runType=NORMAL|WORLD`
- `POST /runs/create`
- `GET /runs/:runId`
- `POST /runs/:runId/move`
- `POST /runs/:runId/reroll`
- `POST /runs/:runId/teleport`

Some endpoints are feature-gated by public host. Treat a not-found or unsupported response as host capability information, not as proof that the session is broken.

## Run Lifecycle

### Get active run

`GET /runs/active`

Use this before creating a new run.

For explicit run type:

- `GET /runs/active?runType=NORMAL`
- `GET /runs/active?runType=WORLD`

### Create run

`POST /runs/create`

Normal run:

```json
{
  "keysAmount": 1,
  "runType": "NORMAL"
}
```

World run, where publicly available:

```json
{
  "keysAmount": 1,
  "runType": "WORLD"
}
```

Behavior:

- `keysAmount` must be an integer `>= 1`
- if `runType` is omitted, the public server may default to normal run behavior
- if a run already exists, the server can return `409 RUN_ALREADY_EXISTS`
- World runs use World-key resources and feature-gated endpoints

### Fetch authoritative state

`GET /runs/:runId`

This is the authoritative snapshot. Expect a payload containing `gameState`, often with `canResume`.

## Action Endpoint

All of these actions use `POST /runs/:runId/move`:

- `move`
- `attack`
- `break`
- `pass`
- `upgrade_selected`
- `use_item`
- `discard_item`

Other action endpoints:

- `POST /runs/:runId/reroll`
- `POST /runs/:runId/teleport`

## Exact Action Payloads

### Pass

```json
{
  "action": {
    "type": "pass"
  },
  "expectedTurnNumber": 12
}
```

### Move

```json
{
  "action": {
    "type": "move",
    "direction": "right",
    "targetX": 22,
    "targetY": 7
  },
  "expectedTurnNumber": 12
}
```

### Attack

```json
{
  "action": {
    "type": "attack",
    "direction": "right",
    "targetEnemyId": "enemy_2"
  },
  "expectedTurnNumber": 12
}
```

### Break

```json
{
  "action": {
    "type": "break",
    "direction": "left",
    "targetId": "crate_1"
  },
  "expectedTurnNumber": 12
}
```

### Select upgrade

```json
{
  "action": {
    "type": "upgrade_selected",
    "upgradeId": "treasure_hunter"
  },
  "expectedTurnNumber": 12
}
```

### Use item

For untargeted active items:

```json
{
  "action": {
    "type": "use_item",
    "itemId": "gas_pedal"
  },
  "expectedTurnNumber": 12
}
```

For targeted active items:

```json
{
  "action": {
    "type": "use_item",
    "itemId": "single_shot",
    "targetX": 24,
    "targetY": 7
  },
  "expectedTurnNumber": 12
}
```

### Discard item

```json
{
  "action": {
    "type": "discard_item",
    "itemId": "bomb"
  },
  "expectedTurnNumber": 12
}
```

Discarding an item is free and should not trigger an enemy turn.

### Reroll

`POST /runs/:runId/reroll`

No action wrapper is required. Use it only while upgrade options are pending.

Reroll cost sequence:

- `10`
- `20`
- `30`
- `50`
- `80`
- `130`
- Fibonacci-style continuation

World runs, where enabled, use the World resource cost at one tenth of the listed cost, minimum `1`.

### Teleport

`POST /runs/:runId/teleport`

Use only when the current state indicates the player is at a portal prompt and the user or strategy explicitly wants teleport.

```json
{
  "confirmed": true
}
```

Teleport uses the same cost progression as reroll. `portal_adept` makes the first teleport free and later teleports cheaper.

## Typical Response Shape

Successful action responses commonly include:

```json
{
  "success": true,
  "gameState": {},
  "events": [],
  "isGameOver": false
}
```

Typical error shape:

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "..."
  }
}
```

## Error Codes Seen In Practice

- `RUN_ALREADY_EXISTS`
- `LOCK_CONFLICT`
- `INTERNAL_ERROR`
- `INSUFFICIENT_TREASURE_FOR_REROLL`
- `VALIDATION_ERROR`
- stale-turn or expected-turn mismatch responses

## Action Selection Contract

Before sending an action:

1. Load fresh `gameState`.
2. If `pendingUpgradeOptions` is non-empty, only choose `upgrade_selected` or `reroll`.
3. If using or discarding an item, verify the item exists in `player.items`.
4. For directional actions, derive the adjacent tile from the player's current `(x, y)`.
5. Use `move`, `attack`, or `break` according to the actual tile content.

## Refetch Rules

Refetch `GET /runs/:runId`:

- after any `500`, `409`, or `429` action error
- after a response that lacks `gameState`
- after a stale-turn or expected-turn mismatch
- before choosing a new action if the last response was ambiguous or partial

Never respond to an ambiguous failure by sending `pass` blindly.
