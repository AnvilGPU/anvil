# ANVIL

[![license](https://img.shields.io/badge/license-MIT-50e3c2)](LICENSE) [![node](https://img.shields.io/badge/node-%E2%89%A5%2022-50e3c2)](.node-version) [![dependencies](https://img.shields.io/badge/dependencies-0-50e3c2)](package.json) [![checks](https://img.shields.io/badge/end--to--end%20checks-78-50e3c2)](test)

**[anvilgpu.run](https://anvilgpu.run)** · [App](https://anvilgpu.run/app) · [Docs](https://anvilgpu.run/docs) · [Changelog](CHANGELOG.md) · [Security](SECURITY.md)

Strike the anvil. ANVIL is a GPU cloud that settles on Elysium: a whole NVIDIA GPU billed by the second, or a single image, video or voice billed by the piece. People pay with USDC, HYPE or $ANVIL. Agents pay with an API key or with x402.

## Run it

```bash
git clone https://github.com/AnvilGPU/anvil
cd anvil
ANVIL_TEST=1 TREASURY=0x000000000000000000000000000000000000beef npm start   # http://localhost:8326
npm test                                                                    # 83 end-to-end checks
```

Needs Node 22 or newer and nothing else. `ANVIL_TEST=1` stands in for the GPU provider and the chain, so every flow works locally with no key and no funds.

## What is in the box

| | |
|---|---|
| **Credits** | 1 credit = $1 of compute at listed prices, sold for 0.70 USDC. Deposits are read back from the chain before they count. Unused credits refund at the price paid, in the asset paid with. |
| **Rentals** | SSH and Jupyter on a whole GPU. The meter starts when SSH is ready and stops when you end it. `anvil/auto` picks the cheapest open card with enough memory. A card that never starts is never billed. |
| **Outputs** | 13 image, video and voice models with a price per output. Credits come back if an output fails. |
| **Keys** | Up to 10 API keys per wallet, each with an optional daily spend cap and a stop switch only the wallet can flip. |
| **MCP** | 14 tools at `/mcp` over Streamable HTTP. |
| **x402** | A request with no key answers 402 with a USDC price. The agent signs an EIP-2612 permit for the price, ANVIL checks the signature itself, pulls the USDC and pays the gas. |
| **Receipts** | One per rental and output, hashed hourly into a Merkle root that is written into the data of a transaction. |
| **$ANVIL** | 1,000 held makes 50 images and 2 videos free per day. Credits bought with it are spent first and the tokens behind them go to the dead address each hour. |

## Source map

```
server/evm.js       keccak-256, secp256k1 (recover + RFC 6979 signing), RLP, EIP-1559, EIP-2612, RPC, payment reads
server/engine.js    the ledger: lots of credits, meters, keys, receipts, refunds, the burn
server/provider.js  the GPU and model provider, plus the local stand-in
server/index.js     routes, the x402 flow, the MCP tools
client/             landing page, app and docs, no build step
test/run.js         the end-to-end suite
```

Nothing is imported from npm. The chain code is written out in `server/evm.js` and checked in the test suite against published values.

## Calling it

```bash
# with a key
curl https://anvilgpu.run/api/v1/outputs \
  -H "Authorization: Bearer $ANVIL_KEY" -H "Content-Type: application/json" \
  -d '{"model": "flux-schnell", "prompt": "A lighthouse at dusk"}'

# rent the cheapest GPU with 48 GB
curl https://anvilgpu.run/api/v1/instances \
  -H "Authorization: Bearer $ANVIL_KEY" -H "Content-Type: application/json" \
  -d '{"gpu": "auto", "vram": 48}'

# an agent over MCP
claude mcp add --transport http anvil https://anvilgpu.run/mcp --header "Authorization: Bearer $ANVIL_KEY"
```

Without the Authorization header, `POST /api/v1/outputs` answers 402 and the `PAYMENT-REQUIRED` header carries what to sign: network `eip155:99901`, asset USDC, EIP-712 domain `USDC` version `1`.

## Checking a receipt

`leaf = sha256(id|wallet|kind|ref|what|micro|seconds|ts)`. Fold the proof in, smaller hash first at each step, and you reach the root of that hour. The root is in the transaction data as `anvil:receipts:v1:HOUR:ROOT`. `GET /api/receipts/{id}/proof` returns the path, and the app does the check in the browser.

## Settings

| Variable | Purpose |
|---|---|
| `TREASURY` | Address that receives deposits and x402 payments |
| `TREASURY_SECRET` | 32-byte hex key for that address. Required for x402. Also sends refunds, the hourly root and the burn. Keep a little HYPE on it for gas |
| `RUNPOD_API_KEY` | Connects the GPUs and model endpoints |
| `TOKEN_ADDRESS` | Contract of $ANVIL. Turns on paying with it, free outputs and the burn |
| `RPC_URL` | Elysium RPC, comma-separated if several |
| `DATA_PATH` | The ledger file. Put it on a persistent disk |
| `MARKUP` | Listed rate over provider cost. Default 1.5, floor 1.43 |
| `POD_IMAGE` | Container image a rented GPU boots |
| `SITE_URL`, `X_URL`, `GH_URL` | Public origin and links |
| `ADMIN_KEY` | Opens `/api/admin/*` |

## Contributing and security

See [CONTRIBUTING.md](CONTRIBUTING.md). Report vulnerabilities [privately](https://github.com/AnvilGPU/anvil/security/advisories/new), never in a public issue.

## License

[MIT](LICENSE)
