# Game API

Use this file for run lifecycle, endpoint payloads, action contracts, and response parsing.

## Confirmed REST Endpoints

- `GET /status`
- `GET /runs/active`
- `POST /runs/create`
- `GET /runs/:runId`
- `POST /runs/:runId/move`
- `POST /runs/:runId/reroll`

## Run Lifecycle

### Get active run

`GET /runs/active`

Use this before creating a new run.

### Create run

`POST /runs/create`

```json
{
  "keysAmount": 1
}
```

Observed behavior:

- `keysAmount` must be an integer `>= 1`
- if a run already exists, the server can return `409 RUN_ALREADY_EXISTS`

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

Only rerolls use:

- `POST /runs/:runId/reroll`

## Exact Action Payloads

### Pass

```json
{
  "action": {
    "type": "pass"
  }
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
  }
}
```

### Attack

```json
{
  "action": {
    "type": "attack",
    "direction": "right",
    "targetEnemyId": "enemy_2"
  }
}
```

### Break

```json
{
  "action": {
    "type": "break",
    "direction": "left",
    "targetId": "crate_1"
  }
}
```

### Select upgrade

```json
{
  "action": {
    "type": "upgrade_selected",
    "upgradeId": "treasure_hunter"
  }
}
```

### Reroll

`POST /runs/:runId/reroll`

No extra action wrapper is required.

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

## Action Selection Contract

Before sending an action:

1. Load fresh `gameState`.
2. If `pendingUpgradeOptions` is non-empty, only choose `upgrade_selected` or `reroll`.
3. For directional actions, derive the adjacent tile from the player's current `(x, y)`.
4. Use `move`, `attack`, or `break` according to the actual tile content.

## Refetch Rules

Refetch `GET /runs/:runId`:

- after any `500`, `409`, or `429` action error
- after a response that lacks `gameState`
- before choosing a new action if the last response was ambiguous or partial

