# NodeRails on Monad testnet

NodeRails is payment infrastructure for crypto commerce: hosted checkout, payment links, invoices, subscriptions, payouts, and disputes. Funds move through on-chain escrow with explicit capture, refund, and settlement rules.

This document focuses on the **Monad testnet** rail (chain ID `10143`) and how the platform, contracts, and ops services fit together.

## Monad testnet deployment

| Field | Value |
|-------|-------|
| Network | Monad testnet |
| Chain ID | `10143` |
| RPC | `https://testnet-rpc.monad.xyz` |
| Explorer | `https://testnet.monadvision.com` |
| Native gas | MON |
| Escrow | `0x5a4bc03b982a909fc1e665cf63c5e662ee74cb2a` |
| Merchant Manager | `0x2fa8ec8170e74afa89991e44e7d8c984f7867e86` |

Solidity sources and deploy scripts live under `noderails/noderails-contracts/`. Addresses are also recorded in `noderails/noderails-contracts/deployData/contractAddresses.json`.

## Mera passkey receiving wallets

Merchants can derive **receiving wallets** from a Monad Foundation **Mera** passkey in the merchant dashboard (`apps/dashboard`). One passkey can derive the EVM path used on Monad and other EVM chains, with optional Solana derivation where enabled. Payout wallets remain separate. Passkey material stays in the browser; the API only stores addresses the merchant saves.

## Architecture

```
Shopper / merchant backends
        |
        v
  NodeRails API + payment UI + dashboard
        |
        +---- BPC --------> balances and FX-style quotes
        |
        +---- MTXM -------> sign + broadcast on Monad
        |
        +---- Indexer ----> escrow / merchant manager events
        |
        v
  Escrow + Merchant Manager (Monad testnet)
```


## MTXM (Multichain Transaction Manager)

Folder: `MultichainTxManager/`

MTXM handles sign, broadcast, and confirmation for on-chain steps that merchants should not run from the API process. NodeRails calls MTXM for escrow capture, settle, refund, dispute resolution, and merchant manager payouts. This tree ships the service layout and API types; the public entrypoint is a reference listener until you configure signers in your own deployment.

## Indexer

Folder: `Indexer/`

Indexer watches registered contracts and programs, normalizes events, and delivers HMAC webhooks into NodeRails. MTXM reports what was submitted; Indexer reports what the chain emitted. Together they give reconciliation finance and support can trust.

## BPC (balance and price check)

Folder: `noderails-bpc-service/`

BPC aggregates RPC endpoints, token metadata, live balances, and USD prices for checkout and dashboards. NodeRails uses it so payment UI and merchant screens do not each maintain their own RPC and price soup.


## Local runbook (outline)

1. Copy `.env.example` files for `noderails`, BPC, and databases; set Postgres, Redis, and JWT secrets.
2. `cd noderails && pnpm install`
3. `pnpm db:migrate:deploy` (or `db:push` for a scratch DB)
4. Start BPC (`cd noderails-bpc-service && pnpm install && pnpm start:local` when configured)
5. Start MTXM and Indexer reference listeners or your full deployments
6. `pnpm dev` in `noderails` for API + payment UI + dashboard

## Demo flow

1. Create a merchant app in the dashboard and enable Monad testnet in admin chain config.
2. Create a payment link or checkout session for a small USDC (or configured token) amount.
3. Open the hosted payment UI, connect a wallet on chain `10143`, and authorize.
4. Confirm escrow state in the dashboard (authorized, captured, settled) and matching indexer webhooks when Indexer is wired.

Contact: **mohit@noderails.com** for partnership or a live walkthrough.
