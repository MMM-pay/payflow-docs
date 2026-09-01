# Contract reference

Three contracts. Dependencies point one way only: `subscription` calls
`plan-registry` and `vault`; neither of those calls anything else in the suite.

## plan-registry

Owns the catalogue of merchant plans.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `initialize` | `admin: Address` | `()` | none (once only) | — |
| `create_plan` | `merchant: Address`, `token: Address`, `amount: i128`, `period: u64` | `u64` | `merchant` | `PlanCreated` |
| `set_plan_active` | `merchant: Address`, `plan_id: u64`, `active: bool` | `()` | `merchant`, must own plan | `PlanStatusChanged` |
| `get_plan` | `plan_id: u64` | `Plan` | none | — |
| `merchant_plans` | `merchant: Address` | `Vec<u64>` | none | — |
| `admin` | — | `Address` | none | — |

`create_plan` rejects `amount <= 0` and any `period` below `MIN_PERIOD`
(60 seconds).

Deactivating a plan stops **new** subscriptions. It does not cancel mandates
already open against it.

## vault

Holds subscriber funds. Custody is deliberately separate from billing so the two
can be audited independently.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `initialize` | `admin: Address` | `()` | none (once only) | — |
| `set_subscription` | `subscription: Address` | `()` | `admin` | — |
| `deposit` | `user: Address`, `token: Address`, `amount: i128` | `()` | `user` | `Deposit` |
| `withdraw` | `user: Address`, `token: Address`, `amount: i128` | `()` | `user` | `Withdraw` |
| `debit` | `user: Address`, `token: Address`, `to: Address`, `amount: i128` | `()` | the subscription contract | `Debit` |
| `balance` | `user: Address`, `token: Address` | `i128` | none | — |
| `subscription` | — | `Address` | none | — |

`debit` calls `require_auth()` on the stored subscription address. That succeeds
because a contract's direct call to another contract is implicitly authorized by
the host. No other caller can satisfy it.

Balances are never locked. `withdraw` always succeeds up to the full balance,
which is how a subscriber starves a mandate they no longer want to fund.

## subscription

The billing engine.

| Function | Parameters | Returns | Auth | Events |
|---|---|---|---|---|
| `initialize` | `admin`, `plan_registry`, `vault`: `Address`, `fee_bps: u32`, `fee_to: Address` | `()` | none (once only) | — |
| `subscribe` | `subscriber: Address`, `plan_id: u64`, `max_charges: u32` | `u64` | `subscriber` | `Subscribed` |
| `charge` | `mandate_id: u64` | `()` | **none — permissionless** | `Charged`, maybe `MandateCompleted` |
| `cancel` | `subscriber: Address`, `mandate_id: u64` | `()` | `subscriber` | `Cancelled` |
| `set_paused` | `subscriber: Address`, `mandate_id: u64`, `paused: bool` | `()` | `subscriber` | `PauseChanged` |
| `get_mandate` | `mandate_id: u64` | `Mandate` | none | — |
| `is_due` | `mandate_id: u64` | `bool` | none | — |
| `subscriber_mandates` | `subscriber: Address` | `Vec<u64>` | none | — |
| `merchant_mandates` | `merchant: Address` | `Vec<u64>` | none | — |
| `fee_bps` | — | `u32` | none | — |

`initialize` rejects `fee_bps > MAX_FEE_BPS` (1000 = 10%).

`max_charges` of `0` means open-ended.

`is_due` reports schedule and status only. It does **not** check whether the
vault can cover the charge.

## Types

```rust
pub struct Plan {
    pub id: u64,
    pub merchant: Address,
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
    pub status: MandateStatus,
}
```

## Events

All events use the `#[contractevent]` macro. Topic 0 is the snake_case event
name; `#[topic]` fields follow; remaining fields are a map in the data section.

| Event | Topics | Data |
|---|---|---|
| `PlanCreated` | `merchant`, `plan_id` | `token`, `amount`, `period` |
| `PlanStatusChanged` | `merchant`, `plan_id` | `active` |
| `Deposit` | `user`, `token` | `amount`, `balance` |
| `Withdraw` | `user`, `token` | `amount`, `balance` |
| `Debit` | `user`, `token`, `to` | `amount`, `balance` |
| `Subscribed` | `mandate_id`, `subscriber`, `merchant` | `plan_id`, `amount`, `period`, `next_charge` |
| `Charged` | `mandate_id`, `subscriber`, `merchant` | `amount`, `fee`, `charges_made`, `next_charge` |
| `Cancelled` | `mandate_id`, `subscriber` | `charges_made` |
| `PauseChanged` | `mandate_id`, `subscriber` | `paused` |
| `MandateCompleted` | `mandate_id`, `subscriber` | `charges_made` |

Note that `Charged.amount` is the **merchant's** share, net of `fee`. Gross is
`amount + fee`.
