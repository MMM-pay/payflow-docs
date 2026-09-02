---
title: The mandate lifecycle
nav_order: 2
---

# The mandate lifecycle

A mandate is the core object in Payflow. Everything else exists to create,
settle, or terminate one.

## States

```
                    subscribe
                        │
                        ▼
    ┌──────────────► Active ──────────────┐
    │                 │  │                │
    │  set_paused     │  │  charge        │  cancel
    │  (false)        │  │  (max reached) │
    │                 ▼  ▼                ▼
    └────────────── Paused            Completed / Cancelled
                        │                (terminal)
                        │ cancel
                        ▼
                    Cancelled
```

| State | Chargeable | How it is reached | Reversible |
|---|---|---|---|
| `Active` | yes, when due | `subscribe`, or resuming from `Paused` | — |
| `Paused` | no | subscriber calls `set_paused(true)` | yes |
| `Cancelled` | no | subscriber calls `cancel` | **no** |
| `Completed` | no | `charges_made` reaches `max_charges` | **no** |

Both terminal states are permanent. A subscriber who wants to start again opens
a new mandate.

## Fields frozen at subscribe time

When `subscribe` runs, it reads the plan once and copies these into the mandate:

- `merchant`
- `token`
- `amount`
- `period`

They are never read from the registry again. This is the guarantee that a
merchant cannot edit a plan and silently reprice existing subscribers. If a
merchant wants new pricing, they publish a new plan, and existing subscribers
stay on the old terms until they choose otherwise.

## Scheduling

`subscribe` sets `next_charge` to the current ledger timestamp, so the first
charge is due immediately.

Each successful `charge` sets:

```
last_charge = now
next_charge = now + period
```

Note that it is `now + period`, **not** `next_charge + period`.

This matters when a keeper goes offline. With `next_charge + period`, five
missed periods would leave five chargeable backlogged charges that could be
collected in a single burst, draining a vault the subscriber thought was safe.
With `now + period`, a keeper outage simply delays billing: at most one charge
is ever collectable at a time.

The tradeoff is slow forward drift — a monthly plan settled a day late bills a
day later every month thereafter. That drift favours the subscriber, which is
the correct direction for an error to point.

## Why charging is permissionless

`charge` takes no authorization. Anyone can call it on any mandate.

That is safe because the mandate already encodes the entire authorization: who
pays, who is paid, how much, and how often. A caller cannot change any of it.
The only thing a caller supplies is the transaction fee.

The benefit is that billing has no single point of failure. Payflow runs a
keeper as a convenience, but if it stops, merchants can settle their own
mandates from the dashboard, and so can subscribers. No user funds are stranded
by an operator going away.
