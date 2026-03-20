# Key Operations

Use this file for buying keys, checking key balance, or sending keys to another address.

## REST Endpoints

- `GET /keys/balance`
- `POST /keys/send`

## Onchain Purchase

Buying keys is not a REST call. It is an onchain AGW transaction.

Confirmed contract data:

- Chain ID: `2741`
- Contract: `0xBDE2483b242C266a97E39826b2B5B3c06FC02916`
- Function: `buyKeys(uint256 quantity)`
- Price per key: `0.001 ETH`

## Buy Keys With AGW

### Function call

- Contract call: `buyKeys(quantity)`
- Native value: `0.001 ETH * quantity`

### Minimal ABI

```json
[
  {
    "type": "function",
    "stateMutability": "payable",
    "name": "buyKeys",
    "inputs": [{ "name": "quantity", "type": "uint256" }],
    "outputs": []
  }
]
```

### Recommended sequence

1. Ensure the AGW tool is connected to chain `2741`.
2. Send the `buyKeys` transaction with the correct payable value.
3. Wait for confirmation.
4. Call `GET /keys/balance` to confirm the balance increased.

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

## Send Keys

`POST /keys/send`

This is an authenticated REST call that requires the MOG session cookie.

Confirmed payload:

```json
{
  "toAddress": "0x...",
  "quantity": 1
}
```

Recommended headers:

- `Accept: */*` or `application/json`
- `Content-Type: application/json`
- `Cookie: <session cookie>`
- `Origin: https://mog.onchainheroes.xyz`
- `Referer: https://mog.onchainheroes.xyz/`

## Send Keys Guardrails

- Validate that `toAddress` is a real EVM address before sending.
- Validate that `quantity` is a positive integer.
- Prefer one send per intended recipient unless the user explicitly wants bulk gifting.
- If the send fails, inspect the response body before retrying.

## Practical Fallbacks

- `GET /keys/balance` failed with `500`: try the intended downstream action once if the user expects balance to be sufficient.
- Onchain buy succeeded but balance still looks stale: refetch once after a short delay.

