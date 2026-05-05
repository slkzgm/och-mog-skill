# Rewards And Pools

Use this file for weekly claims, jackpot claims, weekly pool inspection, and jackpot pool inspection.

## Confirmed Endpoints

If using the standard API base for a public host, use the path forms below.

Read/status endpoints:

- `GET /claims`
- `GET /weekly-pool`
- `GET /jackpot/pool`
- `GET /jackpot/balance`
- `GET /public/pools`

Weekly claim write flow:

- `POST /claims/generate-signatures`
- onchain `claimWeekly(...)`
- `POST /claims/process-claim`

Jackpot claim write flow:

- `POST /jackpot/claim`
- onchain `claimJackpot(...)`
- `POST /jackpot/process-claim`

Claim vault addresses are intentionally not embedded in this skill. Resolve them from the public app/wallet transaction context or a trusted public contract registry for the selected public host before sending a transaction.

## Weekly Claim Status

`GET /claims` returns weekly claim context such as:

- `weekNumber`
- `weekStart`
- `weekEnd`
- `userTreasure`
- `totalTreasure`
- `pool`
- `projectedPayout`
- `totalClaimable`
- `claimableWeeks`

Behavior:

- it is safe to call repeatedly
- `userTreasure` and `projectedPayout` can change as runs finish
- when no claim is available, `totalClaimable` may be `"0"` and `claimableWeeks` may be empty
- weekly claims do not expire once earned

Product-level reward model:

- weekly treasure determines the player's share of the weekly pool
- payout formula: `weekly_pool * (your_treasure / total_treasure)`

## Weekly Claim Execution

### 1. Read status

`GET /claims`

If `claimableWeeks` is empty or `totalClaimable` is zero, do not send a claim transaction.

### 2. Generate signatures

`POST /claims/generate-signatures`

Expected request:

```json
{
  "weeks": [1, 2]
}
```

The server returns signed claim data for the requested weeks.

### 3. Send onchain claim

Send `claimWeekly` with the signed weekly claim entries returned by the server. Do not invent amounts or signatures.

Typical entry shape:

```json
{
  "week": 1,
  "amount": "1000000000000000000",
  "signature": "0x..."
}
```

### 4. Process the transaction

`POST /claims/process-claim`

```json
{
  "txHash": "0x..."
}
```

Then refetch `GET /claims`.

## Jackpot Status

`GET /jackpot/balance` returns pending claimable jackpot value, commonly as:

- `pendingJackpotWei`

When no jackpot is claimable, it may be `"0"`.

`GET /jackpot/pool`, `GET /weekly-pool`, and `GET /public/pools` are lobby/pool inspection endpoints.

## Jackpot Claim Execution

### 1. Read balance

`GET /jackpot/balance`

If no pending balance exists, stop.

### 2. Request claim data

`POST /jackpot/claim`

The server returns signed jackpot claim data when a pending claim exists. If no pending jackpot exists, expect a bad-request style error.

Do not call the onchain claim function with an empty claims array.

### 3. Send onchain claim

Send `claimJackpot` with the signed claim entries returned by the server.

Typical entry shape:

```json
{
  "nonce": "1",
  "amount": "1000000000000000000",
  "signature": "0x..."
}
```

### 4. Process the transaction

`POST /jackpot/process-claim`

```json
{
  "txHash": "0x..."
}
```

Then refetch `GET /jackpot/balance`.

## Pool Contributions

Normal run creation contributes key value into public pools:

- `60%` weekly pool
- `30%` jackpot pool
- remaining value follows product allocation outside these two displayed pools

Purchases alone do not directly credit the weekly or jackpot pool. Pool movement is tied to normal run creation/spend.

World runs, where publicly available, use separate World resources and should not be assumed to fund normal weekly or jackpot pools.

## Jackpot Gameplay Notes

- Sir Jackalot is the jackpot enemy.
- The jackpot payout is credited to pending claimable balance when the server records a qualifying hit/defeat event.
- Jackpot tier percentages currently used by public gameplay before any balance multiplier are:
  - minor: `0.1%` of jackpot pool
  - major: `0.5%` of jackpot pool
  - top tier: `2%` of jackpot pool
- Balance multipliers can reduce displayed payout when the backing claim balance is low.

## Marble Notes

Product-level rules:

- marbles are redeemed weekly
- weekly marble cap is `1000`
- extra marbles may still appear after cap, but are not counted toward reward

## Guardrails

- Use server-generated signatures only.
- Do not reuse signatures across hosts, wallets, or users.
- Do not proceed if the wallet address differs from the authenticated user.
- If an onchain transaction succeeds but REST processing fails, preserve the tx hash and retry processing once before reporting the error.
