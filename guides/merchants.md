# For merchants

## What Payflow gives you

The ability to bill a Stellar customer repeatedly without them signing each
time, and without you holding their keys or their money.

## What it does not give you

A guarantee of payment. A mandate means you *may* collect what was agreed — not
that the funds will be there. If a subscriber's vault is empty, the charge
fails. Treat it like a declined card: retry, notify, and dun.

## 1. Publish a plan

Go to **Merchant**, set a price and a billing period, and publish.

Choose carefully. **Price and period are frozen into every mandate opened
against that plan.** You cannot edit them afterwards — by design, so subscribers
know what they agreed to.

To change pricing, publish a new plan and deactivate the old one. Existing
subscribers stay on their original terms until they move.

## 2. Share it

Subscribers open a mandate from the **Subscribe** page. Each mandate is
independent: cancelling one does not affect the others.

## 3. Get paid

A charge settles when someone calls `charge` on a due mandate. That can be:

- **The Payflow keeper**, automatically. The default.
- **You**, from the merchant dashboard. Any mandate showing as due has a
  **Charge now** button.
- **Anyone else.** Charging is permissionless.

You are never dependent on us being online to collect.

## 4. Track revenue

The merchant dashboard shows:

- **Collected** — total received, net of the protocol fee.
- **Active mandates** — how many subscribers are currently billable.
- **Charges settled** — lifetime successful charges.

## Fees

The protocol takes **1%** on the current testnet deployment, capped in the
contract at 10%. On a 10 XLM charge you receive 9.9 XLM.

## Deactivating a plan

Deactivating stops **new** subscriptions. It does **not** cancel existing
mandates — those keep billing normally. Only the subscriber can end their own
mandate.

If you are shutting a product down, deactivate the plan and tell your
subscribers to cancel. You cannot cancel for them.
