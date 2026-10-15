---
title: Stellar resources
nav_order: 13
---

# Stellar resources

A short set of starting points for readers new to the Stellar network. Payflow
sits on top of three Stellar building blocks you do not need to implement
yourself: the base ledger (accounts, assets, fees), the Soroban smart-contract
runtime, and the off-chain services that expose them. Most of the time this
document is enough; the links are the places to go when you are not.

## Accounts, assets, and fees

- [Stellar Protocol specification](https://github.com/stellar/stellar-protocol)
  - the canonical definitions of accounts, assets, and the fee model. Read
    this first; Payflow's economics page ([`protocol/economics.md`](protocol/economics.md))
    assumes you know what a ledger and a stroop are.
- [Stellar Development Documentation](https://developers.stellar.org/docs)
  - operator and developer reference: how to send payments, fund a test account
    with Friendbot, and read your balance.

## Soroban (smart contracts)

Payflow's mandates live in Soroban contracts. These cover the language and
the runtime you will be reading the contract reference against
([`reference/contracts.md`](reference/contracts.md)):

- [Soroban learning material](https://developers.stellar.org/docs/learn-smart-contracts)
  - Soroban's own tutorial track, from a hello-world contract to production.
- [Rust Soroban SDK](https://github.com/stellar/rs-soroban-sdk)
  - the SDK the Payflow contracts are written in; its doc comments are the
    closest thing to an API reference for what the contracts call.
- [Soroban examples](https://github.com/stellar/soroban-examples)
  - small, runnable contracts for reading state, defining functions, and
    wiring up auth. Good to read before debugging a mandate flow.
- [Soroban](https://soroban.stellar.org)
  - the Soroban home page, linking out to the docs, the explorer, and the
    testnet faucet.

## Networks, services, and tooling

- [Stellar Core / Horizon (the Go SDK)](https://github.com/stellar/go)
  - the reference services behind the Horizon REST API that Payflow's backend
    talks to. The REST endpoint surface is documented in
    [`developers/api.md`](developers/api.md).
- [Soroban developer tooling](https://soroban.stellar.org)
  - the `soroban` CLI for deploying, funding, and inspecting contracts on
    testnet; that is where you point Payflow's `deployments/testnet.env`.

> Testnets, not mainnet: everything above has a testnet counterpart. Do not
> put mainnet contract ids or funded accounts in a PR; link to
> `deployments/testnet.env` instead.
