---
title: Contract reference
nav_order: 7
---

# Contract reference

Three contracts. Dependencies point one way only: `subscription` calls
`plan-registry` and `vault`; neither of those calls anything else in the suite.
This page describes v0.2.0.

## Construction

Every contract is configured by its constructor, which runs in the same
transaction that deploys it. There is no `initialize` function, so there is no
window between deploy and setup in which someone else could claim a contract.

| Contract | Constructor arguments |
|---|---|
| `plan-registry` | `admin: Address` |
| `vault` | `admin: Address` |
| `subscription` | `admin`, `plan_registry`, `vault`: `Address`, `fee_bps: u32`, `fee_to: Address` |

The subscription constructor fails if `fee_bps > MAX_FEE_BPS` (1000 = 10%).
After both exist, the vault admin calls `set_subscription` to name the one
contract allowed to debit the vault. `scripts/deploy.sh` does all of this.

## Paged lists

Lists that grow without bound are stored one ledger entry per position plus a
count, never as a single growing vector. That keeps `subscribe` and
`create_plan` at a constant cost, and stops anyone from blocking a merchant by
opening throwaway mandates until a list outgrows the ledger's entry size limit.

Each list has a `*_count` function and a getter that takes `start` and `limit`
and returns ids oldest first. A page holds at most **50** ids (`MAX_PAGE`),
because one transaction may touch at most 100 ledger entries. To read the
newest entries, read the count and then the last page:

```
count = merchant_mandate_count(merchant)          // e.g. 130
merchant_mandates(merchant, count - 50, 50)       // positions 80..129
```

## plan-registry

Owns the catalogue of merchant plans.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `create_plan` | `merchant: Address`, `token: Address`, `amount: i128`, `period: u64`, `name: String` | `u64` | `merchant` | `PlanCreated` |
| `set_plan_active` | `merchant: Address`, `plan_id: u64`, `active: bool` | `()` | `merchant`, must own plan | `PlanStatusChanged` |
| `get_plan` | `plan_id: u64` | `Plan` | none | — |
| `merchant_plan_count` | `merchant: Address` | `u32` | none | — |
| `merchant_plans` | `merchant: Address`, `start: u32`, `limit: u32` | `Vec<u64>` | none | — |
| `next_plan_id` | — | `u64` | none | — |
| `admin` | — | `Address` | none | — |

`create_plan` rejects `amount <= 0`, any `period` below `MIN_PERIOD`
(60 seconds), and any `name` longer than `MAX_NAME_LEN` (64 **bytes**, not
characters — a multi-byte name can exceed the limit well under 64 characters).
An empty name is valid; clients fall back to displaying the plan id.

Plan ids are sequential from 1, so `next_plan_id() - 1` is the number of plans
published, and a client can list the newest plans without an indexer.

Deactivating a plan stops **new** subscriptions. It does not end mandates
already open against it; the merchant uses `end_mandate` for that.

## vault

Holds subscriber funds. Custody is deliberately separate from billing so the two
can be audited independently.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `set_subscription` | `subscription: Address` | `()` | `admin` | — |
| `deposit` | `user: Address`, `token: Address`, `amount: i128` | `()` | `user` | `Deposit` |
| `withdraw` | `user: Address`, `token: Address`, `amount: i128` | `()` | `user` | `Withdraw` |
| `debit` | `user: Address`, `token: Address`, `to: Address`, `amount: i128` | `()` | the subscription contract | `Debit` |
| `balance` | `user: Address`, `token: Address` | `i128` | none | — |
| `subscription` | — | `Address` | none | — |
| `admin` | — | `Address` | none | — |

`debit` calls `require_auth()` on the stored subscription address. That succeeds
because a contract's direct call to another contract is implicitly authorized by
the host. No other caller can satisfy it.

Balances are never locked. `withdraw` always succeeds up to the full balance,
which is how a subscriber starves a mandate they no longer want to fund.

## subscription

The billing engine.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `subscribe` | `subscriber: Address`, `plan_id: u64`, `max_charges: u32` | `u64` | `subscriber` | `Subscribed` |
| `charge` | `mandate_id: u64` | `()` | **none — permissionless** | `Charged`, maybe `MandateCompleted` |
| `cancel` | `subscriber: Address`, `mandate_id: u64` | `()` | `subscriber` | `Cancelled` |
| `end_mandate` | `merchant: Address`, `mandate_id: u64` | `()` | `merchant` of the mandate | `MandateEnded` |
| `set_paused` | `subscriber: Address`, `mandate_id: u64`, `paused: bool` | `()` | `subscriber` | `PauseChanged` |
| `set_fee_bps` | `new_fee_bps: u32` | `()` | `admin` | `FeeChanged` |
| `get_mandate` | `mandate_id: u64` | `Mandate` | none | — |
| `is_due` | `mandate_id: u64` | `bool` | none | — |
| `subscriber_mandate_count` | `subscriber: Address` | `u32` | none | — |
| `subscriber_mandates` | `subscriber: Address`, `start: u32`, `limit: u32` | `Vec<u64>` | none | — |
| `merchant_mandate_count` | `merchant: Address` | `u32` | none | — |
| `merchant_mandates` | `merchant: Address`, `start: u32`, `limit: u32` | `Vec<u64>` | none | — |
| `fee_bps` | — | `u32` | none | — |
| `admin`, `plan_registry`, `vault` | — | `Address` | none | — |

`max_charges` of `0` means open-ended.

`end_mandate` works on an `Active` or `Paused` mandate and sets it to
`Cancelled`, permanently. It can only stop future charges.

`set_fee_bps` applies to mandates opened afterwards. Each mandate keeps the
`fee_bps` it was opened with.

`is_due` reports schedule and status only. It does **not** check whether the
vault can cover the charge.

## Types

```rust
pub struct Plan {
    pub id: u64,
    pub merchant: Address,
    pub name: String,
    pub token: Address,
    pub amount: i128,
    pub period: u64,
    pub active: bool,
}

pub enum MandateStatus { Active, Paused, Cancelled, Completed }

pub struct Mandate {
    pub id: u64,
    pub subscriber: Address,
    pub plan_id: u64,
    pub merchant: Address,
    pub token: Address,
    pub amount: i128,
    pub period: u64,
    pub next_charge: u64,
    pub last_charge: u64,
    pub charges_made: u32,
    pub max_charges: u32,
    pub fee_bps: u32,
    pub status: MandateStatus,
}
```

A `MandateStatus` read from contract state decodes as a one-element vector,
for example `["Active"]` from `scValToNative`, not as a bare string.

## Events

All events use the `#[contractevent]` macro. Topic 0 is the snake_case event
name; `#[topic]` fields follow; remaining fields are a map in the data section.

| Event | Topics | Data |
|---|---|---|
| `PlanCreated` | `merchant`, `plan_id` | `token`, `amount`, `period`, `name` |
| `PlanStatusChanged` | `merchant`, `plan_id` | `active` |
| `Deposit` | `user`, `token` | `amount`, `balance` |
| `Withdraw` | `user`, `token` | `amount`, `balance` |
| `Debit` | `user`, `token`, `to` | `amount`, `balance` |
| `Subscribed` | `mandate_id`, `subscriber`, `merchant` | `plan_id`, `amount`, `period`, `next_charge`, `max_charges`, `fee_bps` |
| `Charged` | `mandate_id`, `subscriber`, `merchant` | `amount`, `fee`, `charges_made`, `next_charge` |
| `Cancelled` | `mandate_id`, `subscriber` | `charges_made` |
| `MandateEnded` | `mandate_id`, `merchant` | `subscriber`, `charges_made` |
| `PauseChanged` | `mandate_id`, `subscriber` | `paused` |
| `MandateCompleted` | `mandate_id`, `subscriber` | `charges_made` |
| `FeeChanged` | `admin` | `old_fee_bps`, `new_fee_bps` |

Note that `Charged.amount` is the **merchant's** share, net of `fee`. Gross is
`amount + fee`.
