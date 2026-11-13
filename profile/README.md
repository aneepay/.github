<div align="center">

# AneePay

### Non-custodial crypto payments on Polygon

Accept **USDT, USDC and DAI** straight into your own wallet — **0.5% flat fee**,
**no KYC**, **no custody**.

[**Website**](https://aneepay.com) · [**Documentation**](https://docs.aneepay.com) · [**API reference**](https://api.aneepay.com/redoc) · [**Status**](https://status.aneepay.com)

![Networks](https://img.shields.io/badge/networks-Polygon%20%2B%20Amoy-8247E5?style=flat-square)
![Custody](https://img.shields.io/badge/custody-none-2ea44f?style=flat-square)
![KYC](https://img.shields.io/badge/KYC-not%20required-2ea44f?style=flat-square)

</div>

---

## What we build

AneePay is an open-source payment stack for merchants, CMS shops and AI agents that
want to accept stablecoins over HTTP and on-chain — without handing custody of funds
to a third party.

| Property             | Detail                                                               |
| -------------------- | -------------------------------------------------------------------- |
| **Non-custodial**    | Payer → smart contract → merchant wallet. AneePay never holds funds. |
| **Fee**              | Flat **0.5%**, deducted on-chain by the contract.                    |
| **Assets**           | ERC-20 stablecoins: **USDT**, **USDC**, **DAI**.                     |
| **Networks**         | Polygon PoS mainnet and **Amoy** testnet (sandbox).                  |
| **Checkout**         | Hosted payment page (QR + wallet connect) or fully headless API.     |
| **Webhooks**         | HMAC-SHA256 signed server-to-server notifications.                   |
| **Machine payments** | [x402](https://x402.org) v2 pay-per-request for APIs and AI agents.  |

---

## Projects

| Project                                                                             | What it is                                                               | Stack                                        | License                  |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------- | ------------------------ |
| [**aneepay-x402**](https://github.com/aneepay/aneepay-x402)                         | Merchant-side x402 SDK: turn any ASGI app into a paid API                | Python 3.10+ · httpx · Pydantic · Starlette  | MIT                      |
| [**aneepay-contracts**](https://github.com/aneepay/aneepay-contracts)               | Settlement contracts: universal ERC-20 processor and x402 gateway clones | Solidity ^0.8.28 · OpenZeppelin v5 · Foundry | MIT                      |
| [**aneepay-cms-integrations**](https://github.com/aneepay/aneepay-cms-integrations) | Plugins for WooCommerce, OpenCart 3.x and PrestaShop 1.7/8               | PHP 7.4+                                     | MIT                      |
| [**aneepay-docs**](https://github.com/aneepay/aneepay-docs)                         | The source of truth: integration, API, webhooks, x402, security          | MkDocs Material                              | CC0-1.0                  |
| [**aneepay-status**](https://github.com/aneepay/aneepay-status)                     | Public status page and uptime history for AneePay services               | Upptime · GitHub Actions                     | MIT (code) · ODbL (data) |

---

### 🧩 [aneepay-x402](https://github.com/aneepay/aneepay-x402) — paid APIs for humans and agents

A self-contained Python SDK that implements the **x402 wire v2** (`exact` scheme on
EVM) for the AneePay facilitator. It answers `402 Payment Required` with a
machine-readable offer, verifies an **EIP-3009** `TransferWithAuthorization`
signature, runs your handler, and settles on-chain — **without ever touching a
private key**.

```bash
pip install aneepay-x402
```

```python
from fastapi import FastAPI
from x402_sdk import build_payment_required, require_payment

app = FastAPI()
offers = {"/v1/report": build_payment_required(payments, price="0.05")}

@app.middleware("http")
async def x402_middleware(request, call_next):
    offer = offers.get(request.url.path)
    if offer is None:
        return await call_next(request)
    return await require_payment(payments, offer)(request, call_next)
```

- Wire-v2 schemas vendored and pinned by contract tests
- `httpx` + `pydantic` + `starlette` only — no hidden dependencies
- Per-route pricing, shared facilitator client, typed errors, ADRs in [`docs/adr`](https://github.com/aneepay/aneepay-x402/tree/main/docs/adr)
- **Status: alpha** — API may change before 1.0

---

### ⛓️ [aneepay-contracts](https://github.com/aneepay/aneepay-contracts) — settlement on-chain

Two settlement models with the same **50 bps** protocol fee and the same
`PaymentProcessed` event, so off-chain listeners and webhooks work with both:

| Contract                                           | Model                        | Tokens                   | Initiator            |
| -------------------------------------------------- | ---------------------------- | ------------------------ | -------------------- |
| `PolygonStandardPayment`                           | Universal ERC-20 processor   | Any ERC-20, no whitelist | Payer                |
| `PolygonX402GatewayFactory` + `PolygonX402Gateway` | Per-merchant EIP-1167 clones | One whitelisted token    | Facilitator operator |

Highlights:

- EIP-3009 `transferWithAuthorization` and EIP-2612 / DAI permit settlement paths
- Deterministic clone addresses via CREATE2 (`predictClone`) — one clone per account
- `ReentrancyGuard`, `Pausable`, `SafeERC20`, operator-only settlement, `_disableInitializers()`
- Preview fees with `calculateAmounts(amount)`

---

### 🛒 [aneepay-woocommerce](https://github.com/aneepay/aneepay-cms-integrations) — drop-in CMS plugins

One monorepo, three production plugins, one shared release version:

| Platform                 | Folder                          | Artifact                          |
| ------------------------ | ------------------------------- | --------------------------------- |
| WooCommerce (HPOS-ready) | `plugins/woocommerce/aneepay`   | `dist/wc/aneepay.zip`             |
| OpenCart 3.x             | `plugins/opencart/aneepay`      | `dist/oc/aneepay-<ver>.ocmod.zip` |
| PrestaShop 1.7/8         | `plugins/prestashop/ps_aneepay` | `dist/ps/ps_aneepay.zip`          |

```bash
./scripts/pack.sh all      # lint, validate manifests, package artifacts
```

Each plugin redirects the customer to the hosted AneePay checkout, confirms payment
via signed webhooks, and supports the Amoy sandbox. Local test shops for all three
platforms ship in `docker/`.

---

### 📚 [aneepay-docs](https://github.com/aneepay/aneepay-docs) — the manual

End-to-end documentation built with **MkDocs Material** and published to GitHub Pages:
getting started, payment lifecycle, API and webhooks, CMS integration, x402 protocol
and facilitator API, contract reference, security model, statuses, troubleshooting.

- Read: <https://docs.aneepay.com>
- API: <https://api.aneepay.com/redoc> 
- OpenAPI: <https://api.aneepay.com/openapi.json>

---

### 📈 [aneepay-status](https://github.com/aneepay/aneepay-status) — status in public

Powered by [Upptime](https://upptime.js.org): GitHub Actions probes the API, checkout
and journal every 5 minutes, commits response times to git, opens an issue on
downtime, and deploys the status page to GitHub Pages.

- Status page: <https://status.aneepay.com>
- Monitored: `aneepay.com`, `api.aneepay.com`, `journal.aneepay.com`

---

## Security

- **No custody, no keys.** The SDK and merchant servers never hold private keys.
  Customers sign with their own wallets; settlement is broadcast by the facilitator.
- **On-chain fee.** The 0.5% fee is computed and split by the contract, not by a
  backend ledger.
- **Signed webhooks.** Every notification carries an `X-AneePay-Signature`
  HMAC-SHA256 header that must be verified before trusting it.
- **Public security model and disclosure policy** live in the docs:
  [security model](https://github.com/aneepay/aneepay-docs/blob/main/docs/security/security-model.md) ·
  [vulnerability disclosure](https://github.com/aneepay/aneepay-docs/blob/main/docs/security/vulnerability-disclosure.md)

Please do not open public issues for vulnerabilities — follow the disclosure policy.

---

## Contributing

Issues and pull requests are welcome in every repository. Before contributing:

1. Read the project README and its ADRs / roadmap.
2. Keep changes consistent with the existing layering and style.
3. Run the project's lint and test suite (Ruff + pytest, `php -l`, or `forge test`).
4. Describe the **why** in the PR, not only the **what**.

---

<div align="center">

**Built for merchants who would rather keep their own keys.**

[aneepay.com](https://aneepay.com) · [docs.aneepay.com](https://docs.aneepay.com) · [status.aneepay.com](https://status.aneepay.com)

</div>
