---
title: For subscribers
nav_order: 5
---

# For subscribers

## What you are agreeing to

Subscribing creates a **mandate** — a standing permission for one merchant to
collect one fixed amount, no more than once per period.

You are not handing over your wallet. The merchant cannot change the price,
cannot charge early, and cannot charge more often. You can stop it at any time
without asking them.

## Before you start

- A Stellar wallet extension (Freighter is the simplest), set to **Testnet**.
- A funded testnet account — get one at
  [friendbot.stellar.org](https://friendbot.stellar.org).

## 1. Fund your vault

Charges are taken from a vault balance, not straight from your wallet. This is
deliberate: it means a charge can settle while you are offline, and it means the
most a merchant can ever take is what you have deposited.

Go to **My account**, enter an amount, and press **Deposit**.

Deposit roughly what you expect to spend over the next few billing periods. You
can top up whenever, and withdraw whenever.

## 2. Subscribe to a plan

Go to **Subscribe**. Each plan shows its price and billing period.

**Charge limit** caps the total number of times you can ever be charged. Set it
to `3` and the mandate stops itself after three charges. Leave it at `0` for
open-ended.

If you are unsure, set a limit. You can always subscribe again.

The first charge is due immediately.

## 3. Manage it

On **My account**, each subscription can be:

- **Paused** — billing stops, history is kept, and you can resume later.
- **Cancelled** — permanent. To start again you open a new mandate.

Both take effect immediately and neither needs the merchant's agreement.

## How to actually stop paying

You have three independent levers, and any one of them is enough:

1. **Cancel the mandate.** Cleanest.
2. **Pause it.** If you might come back.
3. **Withdraw your vault balance.** Charges then fail for lack of funds.

Nobody can block any of these. Your funds are never locked.

## What can go wrong

**A charge failed.** Almost always an empty vault. Top it up. The merchant is
not paid for a failed charge, and nothing is owed automatically.

**Billing arrived late.** Charges settle when someone triggers them. If the
keeper is down, settlement waits. Delay never stacks up — a keeper offline for
five periods still collects only one charge, not five.

**The merchant changed their price.** They cannot change yours. A price change
means a new plan, and your mandate stays on the terms you agreed to.
