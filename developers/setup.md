---
title: Local setup
nav_order: 9
---

# Local setup

Three repositories. Start with the contracts — the other two need deployed
contract IDs.

## Prerequisites

| Tool | Version | Why |
|---|---|---|
| Rust | 1.96+ | contracts |
| `wasm32v1-none` target | — | `rustup target add wasm32v1-none` |
| Stellar CLI | 27+ | build, deploy, invoke |
| Node.js | 22.5+ | backend uses `node:sqlite` and native TS execution |

## Contracts

```bash
git clone https://github.com/MMM-pay/payflow-contract
cd payflow-contract

cargo test --all          # 42 tests
stellar contract build    # -> target/wasm32v1-none/release/*.wasm
```

Deploy your own suite:

```bash
stellar keys generate payflow-deployer --network testnet --fund
./scripts/deploy.sh testnet payflow-deployer
```

The script deploys in dependency order — registry, then vault, then
subscription — initializes each, grants the subscription contract debit rights
on the vault, and prints an env block. Keep that output.

Smoke-test it end to end:

```bash
REGISTRY=C... VAULT=C... SUBSCRIPTION=C... ./scripts/demo.sh testnet
```

## Backend

```bash
git clone https://github.com/MMM-pay/payflow-backend
cd payflow-backend

npm install
cp .env.example .env      # paste your contract IDs
npm test                  # 20 tests, no network needed
npm run dev
```

Backfill once without running the server:

```bash
npm run indexer
```

The keeper is **off** by default. To turn it on, fund an account with testnet
XLM and set `KEEPER_SECRET` plus `KEEPER_ENABLED=true`. It pays transaction fees
only; it holds no user funds and has no on-chain privilege.

## Frontend

```bash
git clone https://github.com/MMM-pay/payflow-frontend
cd payflow-frontend

npm install
cp .env.example .env.local   # paste the same contract IDs
npm run dev
```

Open http://localhost:3000 and connect a wallet on **Testnet**.

## Environment variables

Contract IDs must match across all three. Get them from `deploy.sh`.

### Backend

| Variable | Default | Purpose |
|---|---|---|
| `SOROBAN_RPC_URL` | `https://soroban-testnet.stellar.org` | RPC endpoint |
| `NETWORK_PASSPHRASE` | `Test SDF Network ; September 2015` | Network id |
| `PLAN_REGISTRY_ID` | — | Registry contract |
| `VAULT_ID` | — | Vault contract |
| `SUBSCRIPTION_ID` | — | Subscription contract |
| `KEEPER_ENABLED` | `false` | Automatic settlement |
| `KEEPER_SECRET` | — | `S...` key paying settlement fees |
| `INDEXER_INTERVAL_MS` | `10000` | Indexing poll |
| `DATABASE_PATH` | `./payflow.db` | SQLite file |
| `PORT` | `8080` | HTTP port |
| `CORS_ORIGIN` | `*` | Allowed origins |

### Frontend

Every variable is `NEXT_PUBLIC_*` and therefore **inlined at build time**.
Changing one requires a rebuild, not a restart. This is the usual reason a
deployed frontend keeps calling `localhost`.

| Variable | Purpose |
|---|---|
| `NEXT_PUBLIC_SOROBAN_RPC_URL` | RPC the browser signs against |
| `NEXT_PUBLIC_NETWORK_PASSPHRASE` | Network id given to the wallet |
| `NEXT_PUBLIC_PLAN_REGISTRY_ID` | Registry contract |
| `NEXT_PUBLIC_VAULT_ID` | Vault contract |
| `NEXT_PUBLIC_SUBSCRIPTION_ID` | Subscription contract |
| `NEXT_PUBLIC_TOKEN_ID` | SEP-41 token for plans |
| `NEXT_PUBLIC_API_URL` | Backend base URL |

## Deployment topology

```
  browser ──── writes (wallet-signed) ────► Soroban RPC ──► contracts
     │                                                          │
     │                                                          │ events
     └──── reads (lists, history) ──► backend ◄─── indexer ─────┘
                                         │
                                      SQLite
```

Frontend on Vercel, backend on Render with a mounted disk for SQLite. The
backend is never in the path of a payment.
