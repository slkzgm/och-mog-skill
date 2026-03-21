---
name: och-mog
description: "Use this skill when an agent needs to operate OCH MOG, Maze of Gain, or OnChainHeroes MOG through wallet signing plus the public REST API: authenticate with SIWE using either an EOA or AGW wallet, preserve the session cookie, buy keys onchain, check key balance, send keys, create or resume runs, inspect game state, execute legal game actions, or inspect claims, weekly pool, and jackpot endpoints."
---

# OCH MOG

Operate MOG through HTTP requests plus standard wallet signing and transaction calls. Do not depend on this repository's CLI or local app at runtime; use them only as the historical source that informed this skill.

## When To Use

Use this skill when the user wants to:

- log into `mog.onchainheroes.xyz` with either an EOA or AGW wallet
- buy keys onchain
- check key balance or send keys to another address
- create, resume, inspect, or play a MOG run through REST
- interpret `gameState`, `mapData`, `fogMask`, enemies, interactives, upgrades, and legal actions
- inspect claims, weekly pool, or jackpot balances

Do not use this skill if the agent lacks both:

- an HTTP-capable tool that can send headers and preserve cookies
- a wallet-capable tool that can sign a SIWE message and send an onchain transaction

## Operating Model

- Default API base: `https://mog.onchainheroes.xyz/api/`
- Treat the server as authoritative for state, rewards, and legality.
- Keep one stable session cookie for the full operation.
- Prefer exact request/response handling over inference.
- After any retryable action failure, refetch state before choosing the next action.

## Quick Start

1. Authenticate with the available wallet.
Read `references/auth-siwe-agw.md`.

2. Choose the operation.
- Keys and gifting: `references/key-ops.md`
- Run lifecycle and endpoint contracts: `references/game-api.md`
- Reading the map and choosing actions: `references/game-state-and-map.md`
- Phase strategy, combat heuristics, and routing priorities: `references/strategy-core.md`
- Upgrade name mapping and known enemy identifiers: `references/upgrade-mapping.md`
- Autoplay policy and backend quirks: `references/strategy-and-gotchas.md`
- Claims, weekly pool, jackpot: `references/rewards-and-pools.md`

3. Preserve the session cookie from `POST /auth/verify` and include it on all later API calls.

## Default Workflow

### Authenticate

1. `GET /auth/nonce`
2. Build the SIWE message with the wallet identity address and the known MOG defaults.
3. Sign the SIWE message with the wallet tool.
4. `POST /auth/verify` with `{ "message": "...", "signature": "0x..." }`
5. Preserve the returned session cookie.
6. `GET /auth/user` to confirm the session is authenticated.

### Buy keys

1. Read `references/key-ops.md`.
2. Use the wallet tool to call `buyKeys(uint256 quantity)` on the known contract.
3. Send `0.001 ETH * quantity`.
4. Confirm with `GET /keys/balance`.

### Start or resume a run

1. `GET /runs/active`
2. If no run is active, `POST /runs/create` with `{ "keysAmount": <n> }`
3. `GET /runs/:runId`
4. Read `references/game-state-and-map.md` before deciding actions
5. If the task is strategic or autoplay-related, also read `references/strategy-core.md` and `references/upgrade-mapping.md`

### Play a turn

1. If `pendingUpgradeOptions` is non-empty, resolve upgrade selection first.
2. Determine the legal action from the adjacent tile and entity state.
3. Use:
- `POST /runs/:runId/move` for `move`, `attack`, `break`, `pass`, `upgrade_selected`
- `POST /runs/:runId/reroll` for rerolls
4. Parse `gameState`, `events`, and `isGameOver`
5. If the action fails with a retryable backend error, refetch with `GET /runs/:runId` before choosing again

### Inspect rewards and pools

1. Read `references/rewards-and-pools.md`.
2. Use the documented GET endpoints first.
3. Do not invent claim POST flows; this skill only treats claims and jackpot payout as confirmed once the write endpoint has been directly observed.

## Non-Negotiable Rules

- Only `pot` and `crate` use explicit `break`.
- `stairs`, `fountain`, `door`, most other interactives, and pickups are movement-based interactions.
- Ghost-like enemies are not valid attack targets.
- If `pendingUpgradeOptions` is non-empty, do not move, attack, break, or pass until the upgrade flow is resolved.
- Never auto-`pass` just because the backend returned `500`.
- On `409`, `429`, or `500` retryable errors, refetch state before repeating or changing the action.
- Treat claims and jackpot payout as read-only discovery areas until a live write endpoint is confirmed.

## Good Defaults

- Send `Accept: application/json` on all API requests.
- Send `Content-Type: application/json` on JSON POST requests.
- Prefer browser-like headers:
  - `Origin: https://mog.onchainheroes.xyz`
  - `Referer: https://mog.onchainheroes.xyz/`
- Use the session cookie returned by auth instead of re-authing before every call.
- Keep decisions grounded in the latest `gameState`, not in stale local assumptions.
