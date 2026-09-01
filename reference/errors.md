# Error codes

Contract errors surface as `Error(Contract, #N)`. The number is scoped to the
contract that raised it, so the same number means different things in different
contracts.

## plan-registry

| # | Name | Meaning |
|---|---|---|
| 1 | `AlreadyInitialized` | `initialize` called twice |
| 2 | `NotInitialized` | Called before `initialize` |
| 3 | `PlanNotFound` | No plan with that id |
| 4 | `NotPlanOwner` | Caller is not the plan's merchant |
| 5 | `InvalidAmount` | Price is zero or negative |
| 6 | `InvalidPeriod` | Period below `MIN_PERIOD` (60s) |

## vault

| # | Name | Meaning |
|---|---|---|
| 1 | `AlreadyInitialized` | `initialize` called twice |
| 2 | `NotInitialized` | Called before `initialize` |
| 3 | `InvalidAmount` | Amount is zero or negative |
| 4 | `InsufficientBalance` | Balance cannot cover the amount |
| 5 | `SubscriptionNotSet` | `set_subscription` has not been called |

## subscription

| # | Name | Meaning |
|---|---|---|
| 1 | `AlreadyInitialized` | `initialize` called twice |
| 2 | `NotInitialized` | Called before `initialize` |
| 3 | `MandateNotFound` | No mandate with that id |
| 4 | `NotSubscriber` | Caller does not own the mandate |
| 5 | `MandateNotActive` | Mandate is paused, cancelled, or completed |
| 6 | `NotDue` | `next_charge` is still in the future |
| 7 | `PlanInactive` | Plan is not accepting new subscribers |
| 8 | `FeeTooHigh` | `fee_bps` above `MAX_FEE_BPS` (1000) |
| 9 | `MaxChargesReached` | Mandate hit its charge cap |
| 10 | `InvalidMaxCharges` | Reserved |

## Common situations

**"Charge fails but the mandate looks fine."** Almost always
`InsufficientBalance` from the vault, not an error from the subscription
contract. Check the subscriber's vault balance.

**"Subscribe fails with `PlanInactive`."** The merchant deactivated the plan.
Existing mandates are unaffected; only new ones are blocked.

**"`NotDue` immediately after subscribing."** The first charge is due at once,
so this normally means it was already settled. Check `charges_made`.
