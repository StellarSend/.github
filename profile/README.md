# StellarSend <img width="42" height="36" alt="stellarsend" src="https://github.com/user-attachments/assets/02614d2d-f331-42ab-88bc-9bcdea60315c" />


**StellarSend** is a non-custodial, global money-transfer platform built entirely on the [Stellar](https://stellar.org) network. It lets anyone send value — XLM, USDC, or any Stellar-based (SEP-41) token — to anyone else, anywhere, with 3–5 second settlement and no intermediary ever holding the funds.

The project is split into three repositories that together form the full stack: on-chain contracts, a Rust API backend, and a React web frontend.

---

## What StellarSend does

- **Instant, global, non-custodial payments** — users sign every transaction with their own wallet (Freighter); private keys never touch StellarSend's servers.
- **Multi-asset transfers** — send XLM, USDC, or any SEP-41 token on Stellar.
- **Path payments** — automatic routing through the Stellar DEX so a sender can pay in one asset and the recipient receives another, with live slippage/price-impact quotes.
- **Protocol fee model** — a small basis-point fee is deducted at the smart-contract level and routed to a dedicated fee-collection contract, auditable on-ledger.
- **Testnet/Mainnet support** — the same stack runs against Stellar testnet (for development) or mainnet (for production) with a one-click toggle.

## How the pieces fit together

```
┌────────────────────┐        ┌───────────────────────┐        ┌─────────────────────────┐
│      frontend       │  API   │        backend        │ Horizon│        contracts         │
│  React + Vite +     │◄──────►│  Rust (axum) + Postgres│◄──────►│  Soroban / Rust, on     │
│  Freighter wallet    │        │  quotes, history, auth │        │  the Stellar network     │
└────────────────────┘        └───────────────────────┘        └─────────────────────────┘
        │                                                                  ▲
        └───────────────── signs & submits transactions ───────────────────┘
```

- The **frontend** builds and signs transactions client-side with the user's own wallet.
- The **backend** provides supporting services — exchange-rate quotes, transaction history, auth — backed by Postgres, and talks to Stellar Horizon.
- The **contracts** are the on-chain source of truth: they validate, execute, and record every payment directly on the Stellar ledger.

---

## Repositories

### [`contracts`](https://github.com/StellarSend/contracts)

Soroban smart contracts (Rust) that power the protocol on-chain:

| Contract | Role |
|---|---|
| `stellar_send` | Main entry point — direct transfers and DEX path payments, fee deduction, pause/unpause, two-step admin rotation |
| `fee_collector` | Accumulates protocol fees per token, admin withdrawal |
| `token_bridge` | Wraps/unwraps assets to enable cross-asset path payments |

`stellar_send` is the only contract end users interact with; `fee_collector` and `token_bridge` are protocol-managed infrastructure.

### [`frontend`](https://github.com/StellarSend/frontend)

The web application users interact with — React 18 + TypeScript + Vite, styled with Tailwind CSS.

- Connects to the Freighter browser wallet; keys never leave the wallet
- Live balances and transaction history pulled from Horizon
- Send flow with quotes, slippage tolerance, and path-payment support
- Activity charts, paginated transaction history, dark-navy UI

### [`backend`](https://github.com/StellarSend/backend)

Supporting API written in Rust (`axum` + `sqlx` + PostgreSQL).

- Auth (JWT + bcrypt)
- Payment quotes and send orchestration
- Transaction history storage and pagination
- Account balance and exchange-rate endpoints, with Horizon as the source of truth

---

## Getting started

Each repo has its own setup instructions in its README. In short:

```bash
# Contracts
git clone https://github.com/StellarSend/contracts && cd contracts
cargo build

# Backend
git clone https://github.com/StellarSend/backend && cd backend
cp .env.example .env && cargo run

# Frontend
git clone https://github.com/StellarSend/frontend && cd frontend
cp .env.example .env && npm install && npm run dev
```

You'll need the [Freighter wallet extension](https://www.freighter.app) and a funded testnet account (via [Friendbot](https://friendbot.stellar.org)) to try the app end-to-end.
