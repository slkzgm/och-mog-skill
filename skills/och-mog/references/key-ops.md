# Key Operations

Use this file for buying keys, checking key balances, sending keys, handling World keys where available, or inspecting transfer history.

## Public Variant Matrix

- OCH MOG: chain `2741`, native `ETH`, AGW-compatible app flow
- Axie: Den of Mysteries: chain `2020`, `USDC`, standard wallet connector flow

Contract addresses are intentionally not embedded in this skill. Resolve them from the public app/wallet transaction context or a trusted public contract registry for the selected public host before sending funds.

## REST Endpoints

Normal keys:

- `GET /keys/balance`
- `POST /keys/process-purchase`
- `POST /keys/send`
- `GET /keys/transfers`

World-key and item balances, where publicly available:

- `GET /items/world-keys`
- `POST /world-keys/send`
- `GET /world-keys/transfers`
- `GET /items/amber`
- `GET /items/balances`

World-key endpoints are feature-gated. If a public host does not expose them, treat that as unsupported for that host.

## Buying Keys

Buying keys has two steps:

1. send the onchain `buyKeys(uint256 quantity)` transaction
2. after confirmation, process the purchase through REST with the transaction hash

Do not treat the onchain transaction alone as sufficient for the REST balance to update.

## Contract Reads Before Purchase

Always read the contract state if the wallet tool supports it:

- `keyPrice()`
- `paused()`

Use the live `keyPrice()` result instead of hardcoding a price.

## OCH MOG Native-Token Purchase

1. Ensure the wallet is on chain `2741`.
2. Resolve the public key purchase contract for OCH MOG.
3. Read `paused()`; stop if paused.
4. Read `keyPrice()`.
5. Call `buyKeys(quantity)` with native value `keyPrice * quantity`.
6. Wait for confirmation.
7. `POST /keys/process-purchase` with:

```json
{
  "txHash": "0x..."
}
```

8. Confirm with `GET /keys/balance`.

## Axie ERC-20 Purchase

1. Ensure the wallet is on chain `2020`.
2. Resolve the public key purchase contract and ERC-20 token contract for Axie.
3. Read `paused()`; stop if paused.
4. Read `keyPrice()`.
5. Check ERC-20 allowance for the purchase contract.
6. If allowance is insufficient, send `approve(spender, amount)`.
7. Call `buyKeys(quantity)` without native value.
8. Wait for confirmation.
9. `POST /keys/process-purchase` with:

```json
{
  "txHash": "0x..."
}
```

10. Confirm with `GET /keys/balance`.

## Minimal ABIs

Key purchase:

```json
[
  {
    "type": "function",
    "stateMutability": "view",
    "name": "keyPrice",
    "inputs": [],
    "outputs": [{ "name": "", "type": "uint256" }]
  },
  {
    "type": "function",
    "stateMutability": "view",
    "name": "paused",
    "inputs": [],
    "outputs": [{ "name": "", "type": "bool" }]
  },
  {
    "type": "function",
    "stateMutability": "payable",
    "name": "buyKeys",
    "inputs": [{ "name": "quantity", "type": "uint256" }],
    "outputs": []
  }
]
```

ERC-20 approval, only for ERC-20 public variants:

```json
[
  {
    "type": "function",
    "stateMutability": "view",
    "name": "allowance",
    "inputs": [
      { "name": "owner", "type": "address" },
      { "name": "spender", "type": "address" }
    ],
    "outputs": [{ "name": "", "type": "uint256" }]
  },
  {
    "type": "function",
    "stateMutability": "nonpayable",
    "name": "approve",
    "inputs": [
      { "name": "spender", "type": "address" },
      { "name": "amount", "type": "uint256" }
    ],
    "outputs": [{ "name": "", "type": "bool" }]
  }
]
```

If a native-token contract ABI marks `buyKeys` as payable and an ERC-20 contract ABI marks it as nonpayable, prefer the live ABI for the selected public host.

## Check Key Balance

`GET /keys/balance`

Typical success shape:

```json
{
  "balance": 3
}
```

Quirk:

- This endpoint has intermittently returned `500` in practice. If it does, do not assume the balance is zero.

## Send Normal Keys

`POST /keys/send`

Authenticated REST call requiring the session cookie.

Payload:

```json
{
  "toAddress": "0x...",
  "quantity": 1
}
```

On Axie, recipient input may also accept supported name formats. Prefer a resolved EVM address when available.

Guardrails:

- Validate `quantity` is a positive integer.
- Validate or resolve the recipient before sending.
- Prefer one send per intended recipient unless the user explicitly wants bulk gifting.
- If the send fails, inspect the response body before retrying.

## Send World Keys

`POST /world-keys/send`

Payload:

```json
{
  "toAddress": "0x...",
  "quantity": 1
}
```

Use only where the endpoint is publicly available. If the endpoint returns not found or unsupported, report that World keys are not enabled for the selected host.

## Transfer History

- `GET /keys/transfers` for normal key transfer history
- `GET /world-keys/transfers` for World-key transfer history where supported

## Practical Fallbacks

- `GET /keys/balance` failed with `500`: retry once after a short delay, then use downstream state cautiously if the user expects balance to be sufficient.
- Onchain buy succeeded but REST balance is stale: call `POST /keys/process-purchase` again with the same tx hash; the flow should be idempotent.
- If processing still fails, keep the tx hash and report the exact REST error without retrying blindly.
