# StellarSend <img width="42" height="30" alt="stellarsend" src="https://github.com/user-attachments/assets/02614d2d-f331-42ab-88bc-9bcdea60315c" />


**StellarSend** is a non-custodial, global money-transfer platform built entirely on the [Stellar](https://stellar.org) network. It lets anyone send value XLM, USDC, or any Stellar-based (SEP-41) token to anyone else, anywhere, with 3–5 second settlement and no intermediary ever holding the funds.

**What makes it different:** any wallet e.g Freighter, Lobstr, xBull can already do a single non-custodial send. That's table stakes, not a product. StellarSend is built to do the things a plain wallet structurally can't, because the logic lives on-chain in Soroban contracts rather than in a UI wrapper around a wallet:

- **Pay on a schedule without being online.** A plain wallet needs you present, unlocked, and signing at the moment of every payment. StellarSend lets you authorize a recurring transfer once the on-chain contract enforces it via a token allowance and executes it on schedule, with no custodian ever holding your keys or funds in between.
- **Pay many people in one atomic step.** A wallet sends one payment per signature. StellarSend fans one signed transaction out to many recipients - payroll, group bills, revenue splits and it's all-or-nothing, not a loop of individually-fallible sends.
- **Get paid, not just send.** A wallet has no concept of "request money." StellarSend can generate a shareable, QR-codeable invoice that anyone can fulfill, making it a two-sided payment tool instead of a one-way pipe.
- **Hold funds under a condition, not just a destination.** A wallet's only primitive is "send it, it's gone." StellarSend can lock funds in an on-chain escrow that releases only after a time window or an arbiter's decision for deposits, marketplace holdbacks, or milestone payments without trusting a company to hold the money in between.

None of this is a centralized feature layer bolted on top of a wallet integration — it's implemented as public functions in the [`contracts`](https://github.com/StellarSend/contracts) repo, so the guarantees come from the chain, not from trusting StellarSend's servers.

The project is split into three repositories that together form the full stack: on-chain contracts, a Rust API backend, and a React web frontend.
X - https://x.com/stellar_send/

<img width="850" height="500" alt="image" src="https://github.com/user-attachments/assets/140020d2-33fb-48e8-9dc9-7d85c256a5bf" />


## What StellarSend does

- **Instant, global, non-custodial payments** — users sign every transaction with their own wallet (Freighter); private keys never touch StellarSend's servers.
- **Multi-asset transfers** — send XLM, USDC, or any SEP-41 token on Stellar.
- **Path payments** — automatic routing through the Stellar DEX so a sender can pay in one asset and the recipient receives another, with live slippage/price-impact quotes.
- **Protocol fee model** — a small basis-point fee is deducted at the smart-contract level and routed to a dedicated fee-collection contract, auditable on-ledger.
- **Testnet/Mainnet support** — the same stack runs against Stellar testnet (for development) or mainnet (for production) with a one-click toggle.
- **Scheduled & recurring payments** — see above.
- **Split / batch payments** — see above.
- **Payment requests / invoicing** — see above.
- **Escrow / conditional transfers** — see above.

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
| `stellar_send` | Main entry point — direct transfers, DEX path payments, fee deduction, pause/unpause, two-step admin rotation, subscriptions, batch payments, payment requests |
| `fee_collector` | Accumulates protocol fees per token, admin withdrawal |
| `token_bridge` | Wraps/unwraps assets to enable cross-asset path payments |
| `escrow` | Locks funds until a time or arbiter condition releases them |

`stellar_send` and `escrow` are the contracts end users interact with; `fee_collector` and `token_bridge` are protocol-managed infrastructure.

### [`frontend`](https://github.com/StellarSend/frontend)

The web application users interact with — React 18 + TypeScript + Vite, styled with Tailwind CSS.

- Connects to the Freighter browser wallet; keys never leave the wallet
- Live balances and transaction history pulled from Horizon
- Send flow with quotes, slippage tolerance, and path-payment support
- Activity charts, paginated transaction history, dark-navy UI
- Subscriptions, batch send, payment requests (with QR codes), and escrow pages — all signed client-side, same as a regular send

### [`backend`](https://github.com/StellarSend/backend)

Supporting API written in Rust (`axum` + `sqlx` + PostgreSQL).

- Auth (JWT + bcrypt)
- Payment quotes and send orchestration
- Transaction history storage and pagination
- Account balance and exchange-rate endpoints, with Horizon as the source of truth
- Subscription, batch-payment, payment-request, and escrow endpoints, plus a keeper service that submits pre-authorized on-chain actions (e.g. due recurring payments) without ever holding user funds or keys

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
