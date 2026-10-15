---
title: FAQ
nav_order: 7
---

# FAQ

The questions that come up most, kept short. Each answer links to the page that
goes deeper.

## For subscribers

**What happens if the keeper stops running?**

Nothing gets lost. A charge settles when *someone* triggers it — the keeper is
just the default trigger. If the keeper is down, settlement waits, and the
delay does not stack up: a keeper offline for five periods still collects only
one charge, not five. See [the mandate lifecycle](../protocol/lifecycle.md#why-charging-is-permissionless)
for why charging is permissionless.

**Will a charge work while I am offline?**

Yes. Charges are taken from your vault balance, not straight from your wallet.
That is deliberate — it lets a charge settle while you are offline, and it caps
the most anyone can take at what you deposited. See
[for subscribers](subscribers.md).

**Can a merchant raise my price?**

No. Price and billing period are frozen into the mandate at subscribe time and
never re-read. If the merchant wants new pricing, they publish a new plan; your
mandate stays on the terms you agreed to. See
[what can go wrong](subscribers.md#what-can-go-wrong).

**What if my vault is empty when a charge comes due?**

The charge fails. Nothing is owed automatically and the merchant is not paid
for a failed charge — it is the same risk as a declined card. Top up the vault
and it will settle next time it is due. See
[what can go wrong](subscribers.md#what-can-go-wrong).

**How do I actually stop paying?**

You have three independent levers and any one is enough: cancel the mandate,
pause it, or withdraw your vault balance. Nobody can block any of these, and
your funds are never locked. See
[how to actually stop paying](subscribers.md#how-to-actually-stop-paying).

## For merchants

**Can I rely on being paid?**

No. A mandate means you *may* collect what was agreed — not that the funds will
be there. If a subscriber's vault is empty the charge fails. Treat it like a
declined card: retry, notify, and dun. See
[what Payflow does not give you](merchants.md#what-it-does-not-give-you).

**Who can end one of my subscribers' mandates?**

You can, with `end_mandate` from the merchant dashboard. It cancels the mandate
for good and stops future charges — it cannot take money from the subscriber or
refund it. Deactivating a plan stops *new* subscriptions but does not end
existing mandates. See
[ending a subscriber's mandate](merchants.md#ending-a-subscribers-mandate).

**Am I dependent on the Payflow keeper being online to get paid?**

No. Charging is permissionless: the merchant dashboard, or anyone else, can
settle a due mandate. No user funds are stranded if an operator goes away. See
[why charging is permissionless](../protocol/lifecycle.md#why-charging-is-permissionless).

## Fees

**What does the protocol cost?**

The current testnet deployment charges **1%** per settled charge, capped in the
contract at 10% (`MAX_FEE_BPS = 1000`). On a 10 XLM charge the merchant receives
9.9 XLM. The fee is frozen into each mandate at subscribe time. See
[economics — the fee](../protocol/economics.md#the-fee).
