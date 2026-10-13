# NodeRails on Monad

NodeRails is payment infrastructure for crypto commerce. Merchants get hosted checkout, payment links, invoices, subscriptions, payouts, and disputes. Shoppers pay from a wallet. Funds sit in **on-chain escrow** with clear rules for capture, refund, dispute, and settlement.

This repository packages the **Monad testnet** merchant platform (`noderails/`), EVM escrow contracts, and the supporting ops services (**MTXM**, **Indexer**, **BPC**) that production NodeRails runs alongside the API.

Live product: [noderails.com](https://www.noderails.com) · Merchant dashboard: [merchant.noderails.com](https://merchant.noderails.com)

---

## Why Monad

Monad testnet is a high-throughput EVM environment suited to retail checkout: familiar tooling (Foundry, wallets), fast feedback for merchants, and alignment with Monad Foundation wallet programs such as **Mera** passkey receiving wallets in the dashboard.

| | |
|--|--|
| Network | Monad testnet |
| Chain ID | `10143` |
| RPC | `https://testnet-rpc.monad.xyz` |
| Explorer | `https://testnet.monadvision.com` |
| Native gas | MON |
| Escrow | `0x5a4bc03b982a909fc1e665cf63c5e662ee74cb2a` |
| Merchant Manager | `0x2fa8ec8170e74afa89991e44e7d8c984f7867e86` |

---

## The problem we solve

**For shoppers**  
Wallet checkout must show clear amounts, status, and completion. Abandonment spikes when the flow feels like a developer tool.

**For merchants**  
You need payment intents, webhooks, refunds, and reports finance can trust, not one-off wallet addresses.

**For platform ops**  
Someone must sign and broadcast transactions, index contract events, and price tokens without putting hot keys in the main API. NodeRails ships that split explicitly.

---

## What merchants get

- Hosted checkout and payment UI on Monad testnet (and other configured EVM chains)
- Payment links, invoices, one-time and subscription billing
- Payment intents with lifecycle states from created through settled or refunded
- Webhooks and HTTP APIs for your backend
- Merchant dashboard for apps, keys, customers, payouts, and bank settings where enabled
- **Mera passkey receiving wallets** in dashboard settings (browser-held keys; saved addresses only on the server)

---

## How the stack fits together

```
Shopper / merchant apps
        |
        v
   NodeRails API + payment UI
        |
        +---- BPC --------> live prices and balances
        |
        +---- MTXM -------> sign + broadcast on Monad
        |
        +---- Indexer ----> contract events as webhooks
        |
        v
   Escrow + Merchant Manager (Monad testnet)
```

1. Checkout uses **BPC** for consistent balances and quotes.
2. Capture, settle, refund, and payout requests go to **MTXM** (configure signers in your deployment).
3. **Indexer** watches escrow and merchant manager on chain `10143` and posts events into NodeRails.
4. Merchant backends integrate with NodeRails APIs only.

---

## Repository map

```
├── README.md
├── METROPOLIS.md
├── noderails/                  # Apps, API, packages, EVM contracts
├── MultichainTxManager/        # MTXM
├── Indexer/                    # On-chain indexer
└── noderails-bpc-service/      # Prices and balances
```

| Path | Contents |
|------|----------|
| `noderails/apps/payment-ui` | Hosted checkout |
| `noderails/apps/dashboard` | Merchant dashboard (Mera wallets) |
| `noderails/apps/admin` | Platform admin |
| `noderails/services/noderails-server` | Main API |
| `noderails/packages/mtxm-client` | MTXM HTTP client |
| `noderails/packages/indexer-client` | Indexer HTTP client |
| `noderails/noderails-contracts` | Foundry escrow + merchant manager |

---

## Quick start

### Prerequisites

- Node.js 20+
- pnpm 9
- PostgreSQL and Redis reachable from your machine
- Foundry (for contract work on Monad testnet)

### Install

```bash
cd noderails
pnpm install
cp .env.example .env   # create from your secrets manager; never commit .env
pnpm db:migrate:deploy
```

### Run (development)

```bash
# Terminal 1: BPC (after configuring noderails-bpc-service/.env)
cd noderails-bpc-service && pnpm install && pnpm start:local

# Terminal 2: MTXM + Indexer reference listeners or full stacks
cd MultichainTxManager && pnpm install && pnpm --filter backend dev
cd Indexer && pnpm install && pnpm --filter backend dev

# Terminal 3: NodeRails
cd noderails && pnpm dev
```

Docker: see `noderails/docker-compose.yml` for API + Caddy; database and Redis are external.

### Demo

1. Open the merchant dashboard, create an app, connect Monad testnet tokens.
2. Create a payment link; open it in payment UI.
3. Pay with a Monad testnet wallet; watch intent state and escrow on [MonadVision](https://testnet.monadvision.com).

See **METROPOLIS.md** for architecture detail.

---

## Contact

**Martand Rise / NodeRails** · **mohit@noderails.com**
