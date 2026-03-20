# Auth And Session

Use this file when the task involves logging into MOG, re-authing after expiry, or explaining how AGW and the REST session fit together.

## Confirmed Defaults

- API base: `https://mog.onchainheroes.xyz/api/`
- SIWE domain: `mog.onchainheroes.xyz`
- SIWE URI: `https://mog.onchainheroes.xyz`
- SIWE statement: `Sign in with Ethereum to the app.`
- SIWE version: `1`
- Chain ID: `2741`
- Typical expiration window used by our client and CLI: `7` days

## Required Endpoints

- `GET /auth/nonce`
- `POST /auth/verify`
- `GET /auth/user`
- Optional informational endpoint: `GET /profile/:address`

## AGW Model

This skill assumes a standard AGW tool is available.

- The identity used inside the SIWE message should be the AGW account address.
- The message should be signed through the AGW tool, not by manually faking an EOA signature flow.
- After `POST /auth/verify`, preserve the session cookie and reuse it for later REST calls.

## SIWE Message Template

Use this exact structure:

```text
mog.onchainheroes.xyz wants you to sign in with your Ethereum account:
<AGW_ADDRESS>

Sign in with Ethereum to the app.

URI: https://mog.onchainheroes.xyz
Version: 1
Chain ID: 2741
Nonce: <NONCE>
Issued At: <ISO_TIMESTAMP>
Expiration Time: <ISO_TIMESTAMP_PLUS_7_DAYS>
```

Notes:

- `Issued At` must be a valid ISO-8601 timestamp.
- `Expiration Time` should also be ISO-8601.
- The nonce comes from `GET /auth/nonce`.

## Authentication Sequence

### 1. Request nonce

`GET /auth/nonce`

Accept either a direct string nonce or an object containing a `nonce` field.

### 2. Build SIWE message

Use the AGW address and the template above.

### 3. Sign with AGW

Ask the AGW tool to sign the SIWE message exactly as text.

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
- `Origin: https://mog.onchainheroes.xyz`
- `Referer: https://mog.onchainheroes.xyz/`
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
- `401` or `403` after previously working auth: refresh the session once, then continue.
- Missing cookie storage in the agent's HTTP tool: stop and state that the tool cannot complete the MOG login flow safely.

