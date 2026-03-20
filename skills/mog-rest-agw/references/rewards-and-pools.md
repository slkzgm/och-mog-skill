# Rewards And Pools

Use this file for claims, weekly pool, and jackpot inspection.

## Confirmed Read Endpoints

- `GET /api/claims`
- `GET /api/weekly-pool`
- `GET /api/jackpot/pool`
- `GET /api/jackpot/balance`

If you are already using the standard API base `https://mog.onchainheroes.xyz/api/`, the path forms are:

- `/claims`
- `/weekly-pool`
- `/jackpot/pool`
- `/jackpot/balance`

## What We Know About Claims

`GET /claims` has been observed returning weekly claim context such as:

- `weekNumber`
- `weekStart`
- `weekEnd`
- `userTreasure`
- `totalTreasure`
- `pool`
- `projectedPayout`
- `totalClaimable`
- `claimableWeeks`

Observed behavior:

- it is polled frequently by the client
- `userTreasure` and `projectedPayout` change over time
- when no claim is available, `totalClaimable` may be `"0"` and `claimableWeeks` may be empty

## What We Know About Jackpot

`GET /jackpot/balance` has been observed returning:

- `pendingJackpotWei`

When no jackpot is claimable, it may be `"0"`.

`GET /jackpot/pool` and `GET /weekly-pool` have also been observed as read-only lobby endpoints.

## Important Boundary

This skill does not currently treat claim payout or jackpot payout as a confirmed write workflow.

Reason:

- a claim POST endpoint has not yet been directly confirmed in our captured traffic
- a jackpot payout POST endpoint has not yet been directly confirmed either

Do not invent one of these:

- `POST /claims`
- `POST /claims/claim`
- `POST /claims/:week/claim`
- `POST /jackpot/claim`

Only use a claim or jackpot write endpoint once it has been observed live and its payload is known.

## Safe Usage Pattern

1. Authenticate normally.
2. Read the known GET endpoints.
3. If claimable data is non-zero and the user wants payout automation, state that the write endpoint must be freshly discovered from live traffic first.
4. Once a write endpoint is confirmed in the future, add it to this skill rather than improvising.

## Correlation Hint

Historical analysis showed that `userTreasure` on `/claims` can move after `game_over`, which is useful for validation, but it still does not confirm a public client-side claim POST flow.

