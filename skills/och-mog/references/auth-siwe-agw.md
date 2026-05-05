# Auth And Session

Use this file when the task involves logging into a public MOG host, re-authing after expiry, or explaining how wallet signing and the REST session fit together.

## Public Hosts

- OCH MOG API base: `https://mog.onchainheroes.xyz/api/`
- Axie: Den of Mysteries API base: `https://axiedom.xyz/api/`

The request host selects the variant. Do not hardcode one host into a session for another host.

## Chains And Wallet Modes

Mainnet public variants:

- OCH MOG: chain `2741`, native `ETH`, Abstract wallet / AGW flow supported
- Axie: Den of Mysteries: chain `2020`, `USDC`, standard EOA-style wallet connector

Operational notes:

- OCH MOG supports AGW/session-key behavior in the app context.
- Axie uses a normal wallet connector flow; do not assume AGW session keys there.
- Always switch the wallet to the public chain that matches the selected host before signing or sending transactions.

## Required Endpoints

- `GET /auth/nonce`
- `POST /auth/verify`
- `GET /auth/user`
- Optional informational endpoint: `GET /profile/:address`

## SIWE Rules

Use these values dynamically from the selected public host:

- `domain`: exact request host, for example `mog.onchainheroes.xyz` or `axiedom.xyz`
- `uri`: exact origin, for example `https://mog.onchainheroes.xyz`
- `statement`: `Sign in with Ethereum to the app.`
- `version`: `1`
- `chainId`: selected public chain
- `nonce`: value returned by `GET /auth/nonce`

The nonce is scoped by host/variant. If the selected host changes, request a new nonce and rebuild the SIWE message.

## Wallet Model

This skill assumes a standard wallet tool is available.

- The identity address in the SIWE message should be the wallet address that MOG should recognize for the session.
- For an EOA flow, this is the EOA address.
- For an AGW smart account flow, this should be the AGW account address.
- The message should be signed through the wallet tool, not by fabricating signatures manually.
- After `POST /auth/verify`, preserve the session cookie and reuse it for later REST calls on the same host.

## SIWE Message Template

Use this structure, replacing every host/origin/chain value for the selected public host:

```text
<DOMAIN> wants you to sign in with your Ethereum account:
<WALLET_IDENTITY_ADDRESS>

Sign in with Ethereum to the app.

URI: <ORIGIN>
Version: 1
Chain ID: <CHAIN_ID>
Nonce: <NONCE>
Issued At: <ISO_TIMESTAMP>
Expiration Time: <ISO_TIMESTAMP_PLUS_7_DAYS>
```

Notes:

- `Issued At` must be a valid ISO-8601 timestamp.
- `Expiration Time` should also be ISO-8601.
- The nonce comes from `GET /auth/nonce` on the same host that will receive `POST /auth/verify`.

## Authentication Sequence

### 1. Request nonce

`GET /auth/nonce`

The endpoint returns a plain text nonce in current public flows. Tolerate an object containing a `nonce` field if an HTTP helper normalizes the response.

### 2. Build SIWE message

Use the wallet identity address and the template above.

### 3. Sign with the wallet

Ask the wallet tool to sign the SIWE message exactly as text.

### 4. Verify

`POST /auth/verify`

```json
{
  "message": "<full siwe message>",
  "signature": "0x..."
}
```

Expected success shape:

```json
{ "ok": true }
```

The important side effect is the returned session cookie.

### 5. Confirm session

`GET /auth/user`

Typical success shape:

```json
{
  "ok": true,
  "user": {
    "address": "0x...",
    "userId": "...",
    "expirationTime": "..."
  }
}
```

## Header Discipline

Use:

- `Accept: application/json`
- `Origin: https://<public-host>`
- `Referer: https://<public-host>/`
- `Cookie: <session cookie>` on authenticated calls

For JSON POST requests, also use:

- `Content-Type: application/json`

## Cookie Discipline

- Preserve the cookie returned by `POST /auth/verify`.
- Reuse the same cookie for the whole run batch when possible.
- If a later request returns `401` or `403`, re-auth once, then retry the blocked request.
- Do not mix authenticated and unauthenticated calls in the middle of a run unless you intentionally refreshed the session.

## Failure Handling

- Empty nonce: stop and report auth failure.
- `INVALID_DOMAIN` or `INVALID_URI`: rebuild the SIWE message with the exact selected public host and origin.
- `INVALID_CHAIN_ID` or `INVALID_CHAIN`: switch the wallet to the selected public chain and request a new nonce.
- `NONCE_NOT_FOUND` or `NONCE_EXPIRED`: request a new nonce and rebuild the message.
- `401` or `403` after previously working auth: refresh the session once, then continue.
- Missing cookie storage in the agent's HTTP tool: stop and state that the tool cannot complete the login flow safely.

## Choosing The Identity Address

Use this rule:

- if the user is operating through a normal EOA, use the EOA address in the SIWE message
- if the user is operating through an AGW smart account, use the AGW account address in the SIWE message

Everything after authentication is the same REST flow for a given public host.
