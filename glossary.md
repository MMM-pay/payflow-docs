---
title: Glossary
nav_order: 14
---

# Glossary

Terms used across the Payflow docs, each with a one- or two-sentence definition
and a link to the page that explains it in depth. Everything here is grounded
in the v0.2.0 contracts and the surrounding pages.

## Core Payflow terms

- **Mandate** — the standing authorization a subscriber signs once: who pays,
  who is paid, how much, and how often. It is the object everything else in
  Payflow exists to create, settle, or terminate. See
  [the mandate lifecycle](protocol/lifecycle.md).
- **Charge** — one collection of a mandate when it comes due. At most one
  charge is collectable per period, and only while the mandate is `Active`.
  See [the mandate lifecycle](protocol/lifecycle.md).
- **Plan** — a merchant-published price, billing period, and token, held in the
  plan registry. The terms are copied into the mandate at `subscribe` time and
  never re-read, which is why a merchant cannot reprice an existing subscriber.
  See [the mandate lifecycle](protocol/lifecycle.md) and the
  [contract reference](reference/contracts.md).
- **Plan registry** — the public contract that stores plans; immutable except
  for an on/off switch. See the [contract reference](reference/contracts.md).
- **Keeper** — the actor that submits a `charge` when a mandate falls due.
  Charging is permissionless, so this can be Payflow's own keeper, the
  merchant, the subscriber, or anyone — the mandate already carries the full
  authorization. See [the mandate lifecycle](protocol/lifecycle.md).
- **Vault** — the custody contract that holds a subscriber's funds and pays out
  settled charges. The subscriber can withdraw (drain) it at any time, which is
  how they starve a mandate. Custody is a liability, discussed on the
  [trust model](protocol/trust.md).
- **Token** — the asset a mandate pays in, XLM or a SEP-41 token. Amounts are
  always in that token's smallest unit.
- **Period** — the billing interval. Scheduling uses `now + period` so a missed
  charge can never build up a backlogged burst. See
  [the mandate lifecycle](protocol/lifecycle.md).
- **Mandate state** — `Active`, `Paused`, `Cancelled`, or `Completed`; the last
  two are terminal and cannot be reversed. See
  [the mandate lifecycle](protocol/lifecycle.md).
- **`next_charge` / `last_charge`** — the two scheduling timestamps a settled
  charge updates. See [the mandate lifecycle](protocol/lifecycle.md).
- **`fee_bps`** — the protocol fee, in basis points, frozen into the mandate at
  subscribe time. A subscriber always settles at the rate they signed up.
  See [economics](protocol/economics.md).
- **Subscription contract** — the contract that checks the schedule, debits the
  vault, pays the merchant, and takes the protocol fee. See the
  [contract reference](reference/contracts.md).
- **Subscription admin** — the key that sets the protocol fee, capped at 10%.
  A change applies only to mandates opened afterwards. See the
  [trust model](protocol/trust.md).
- **Vault admin** — the key that can repoint `set_subscription` at a different
  contract; the main custody risk in the design. See the
  [trust model](protocol/trust.md).
- **`MIN_PERIOD`** — a 60-second floor on the billing period, set so validator
  timestamp skew cannot meaningfully accelerate billing. See the
  [trust model](protocol/trust.md).
- **SEP-41** — the token standard the vault debits; every code path writes state
  before transferring, so a hostile token can fail a charge but not steal. See
  the [trust model](protocol/trust.md).

## Stellar primitives

- **Stroop** — the smallest unit of XLM; 1 XLM = 10 000 000 stroops. All Payflow
  amounts are integer stroops, never JSON numbers. See
  [economics](protocol/economics.md).
- **Base fee** — the Stellar transaction fee (100 stroops) paid by whoever
  submits a charge. This settlement cost is why Payflow is a Stellar product.
  See [economics](protocol/economics.md).
- **Basis points (bps)** — hundredths of a percent; the protocol fee is
  `amount × fee_bps ÷ 10 000`, capped at `MAX_FEE_BPS = 1000` (10%). See
  [economics](protocol/economics.md).
