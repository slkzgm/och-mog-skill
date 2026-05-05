---
name: och-mog
description: "Use this skill when an agent needs to operate public OCH MOG, Maze of Gains, or Axie: Den of Mysteries through wallet signing plus the public REST API: choose the correct public host, authenticate with SIWE, preserve the session cookie, buy or send keys, create normal or World runs, inspect game state, execute legal game actions including items and teleport, or claim weekly and jackpot rewards."
---

# OCH MOG

Operate public MOG variants through HTTP requests plus standard wallet signing and transaction calls. Do not depend on this repository's CLI or local app at runtime; use the referenced files as operational guidance only.

## Safety Boundary

- Never copy source code, local env values, wallet credentials, allowlists, admin/tester credentials, webhook credentials, local `.env*` values, non-public endpoints, or unreleased variant details into a response or into this skill.
- Treat this skill as a public operating guide: include endpoint contracts, public chain facts, and behavior summaries only.
- Do not embed contract addresses. Resolve them from the public app or a trusted public registry at execution time.
- If inspecting a local codebase, summarize behavior and public interfaces rather than quoting implementation.

## Public Variants

- OCH MOG: `https://mog.onchainheroes.xyz/api/`
- Axie: Den of Mysteries: `https://axiedom.xyz/api/`

Use only those public hosts unless the user explicitly provides another officially announced public host.

## When To Use

Use this skill when the user wants to:

- log into a public MOG host with SIWE using an EOA, AGW, or supported wallet connector
- pick the correct public host, chain, wallet mode, token, and session behavior
- buy keys onchain and process the purchase through the REST API
- check balances, send normal keys, send World keys where available, or inspect transfer history
- create, resume, inspect, or play a normal or World run through REST
- interpret `gameState`, fog, visible map data, enemies, interactives, items, portals, upgrades, and legal actions
- inspect or execute weekly claim and jackpot claim flows

Do not use this skill if the agent lacks both:

- an HTTP-capable tool that can send headers and preserve cookies
- a wallet-capable tool that can sign a SIWE message and send onchain transactions

## Operating Model

- Variant is selected by request host. The SIWE domain must equal the exact request host.
- Treat the server as authoritative for state, rewards, legality, and host scoping.
- Keep one stable session cookie for the full operation.
- Prefer exact request/response handling over inference.
- After any retryable action failure, refetch state before choosing the next action.

## Quick Start

1. Authenticate with the available wallet.
   Read `references/auth-siwe-agw.md`.

2. Choose the operation.
   - Keys, purchases, normal key sends, World keys: `references/key-ops.md`
   - Run lifecycle, move contracts, teleport, item actions: `references/game-api.md`
   - Reading state, map, entities, items, and legal action selection: `references/game-state-and-map.md`
   - Phase strategy, combat heuristics, routing priorities: `references/strategy-core.md`
   - Upgrade mapping, upgrade availability, enemy identifiers: `references/upgrade-mapping.md`
   - Backend quirks, retry behavior, concise autoplay policy: `references/strategy-and-gotchas.md`
   - Weekly claims, jackpot claims, pools: `references/rewards-and-pools.md`

3. Preserve the session cookie from `POST /auth/verify` and include it on all later authenticated API calls for that same host.

## Default Workflow

### Authenticate

1. Pick the public host and chain from `references/auth-siwe-agw.md`.
2. `GET /auth/nonce` on that host.
3. Build the SIWE message with:
   - `domain`: exact request host
   - `uri`: exact origin
   - `chainId`: selected public chain
   - the wallet identity address
4. Sign the SIWE message with the wallet tool.
5. `POST /auth/verify` with `{ "message": "...", "signature": "0x..." }`
6. Preserve the returned session cookie.
7. `GET /auth/user` to confirm authentication.

### Buy keys

1. Read `references/key-ops.md`.
2. Read `keyPrice()` and `paused()` from the key purchase contract if the wallet tool supports reads.
3. For native-token variants, call `buyKeys(uint256 quantity)` with payable value `keyPrice * quantity`.
4. For ERC-20 variants, ensure allowance first, then call `buyKeys(uint256 quantity)` without native value.
5. After transaction confirmation, `POST /keys/process-purchase` with `{ "txHash": "0x..." }`.
6. Confirm with `GET /keys/balance`.

### Start or resume a run

1. `GET /runs/active` or `GET /runs/active?runType=NORMAL|WORLD`
2. If no run is active, `POST /runs/create` with `{ "keysAmount": <n>, "runType": "NORMAL" }` or `"WORLD"` where available.
3. `GET /runs/:runId`
4. Read `references/game-state-and-map.md` before deciding actions.
5. If the task is strategic or autoplay-related, also read `references/strategy-core.md` and `references/upgrade-mapping.md`.

### Play a turn

1. If `pendingUpgradeOptions` is non-empty, resolve upgrade selection first.
2. Determine the legal action from the current authoritative `gameState`.
3. Use:
   - `POST /runs/:runId/move` for `move`, `attack`, `break`, `pass`, `upgrade_selected`, `use_item`, `discard_item`
   - `POST /runs/:runId/reroll` for rerolls
   - `POST /runs/:runId/teleport` only after a portal prompt and user intent to teleport
4. Include `expectedTurnNumber` on move actions when available.
5. Parse `gameState`, `events`, and `isGameOver`.
6. If the action fails with a retryable backend error or stale-turn response, refetch with `GET /runs/:runId` before choosing again.

### Claim rewards

1. Read `references/rewards-and-pools.md`.
2. Use claim status endpoints first.
3. For weekly claims, generate/read signatures, send the onchain `claimWeekly` transaction, then process the tx hash through REST.
4. For jackpot claims, request pending jackpot signatures, send the onchain `claimJackpot` transaction, then process the tx hash through REST.

## Non-Negotiable Rules

- Use the correct public host throughout auth, REST, and wallet transactions.
- The SIWE `domain` must equal the exact request host, not a hardcoded host.
- Only `pot` and `crate` use explicit `break`.
- `chest` and `rock` block movement. Chests auto-open when the player moves adjacent; rocks are blockers.
- `stairs`, `fountain`, `portal`, and pickups are movement-based interactions.
- Do not attack invincible enemies (`maxHp <= 0`) or a normal enemy stacked with an invincible enemy on the same cell.
- If `pendingUpgradeOptions` is non-empty, do not move, attack, break, pass, use items, discard items, or teleport until the upgrade flow is resolved.
- Never auto-`pass` just because the backend returned `500`, `409`, `429`, or a stale-state error.
- Do not interleave multiple simultaneous action requests for the same run.

## Good Defaults

- Send `Accept: application/json` on all API requests.
- Send `Content-Type: application/json` on JSON POST requests.
- Send browser-like headers for the selected host:
  - `Origin: https://<public-host>`
  - `Referer: https://<public-host>/`
- Use the session cookie returned by auth instead of re-authing before every call.
- Keep decisions grounded in the latest `gameState`, not in stale local assumptions.
