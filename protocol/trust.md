# Trust model

## What a subscriber must trust

**The contract code.** It is open source and unaudited. Read it.

**Nothing else.** Specifically, a subscriber does *not* have to trust:

- *The merchant.* Mandate terms are frozen at subscribe time. The merchant
  cannot raise the price, shorten the period, or charge early.
- *The keeper.* It can only trigger a charge the mandate already permits. It
  cannot charge more, sooner, or after cancellation.
- *The Payflow frontend.* Every transaction is built locally and signed in the
  user's wallet. The app never holds a key and cannot move funds.
- *The indexer.* It serves read-only lists. A compromised indexer can display
  wrong information; it cannot cause a payment.

## What a merchant must trust

**That the subscriber keeps their vault funded.** Payflow guarantees a merchant
*may* collect what the mandate permits — not that the money will be there. An
empty vault means the charge fails. This is the same risk as a declined card,
and merchants should handle it the same way.

## What both parties trust the admin with

This is the weakest part of the current design, and it is stated plainly rather
than buried.

The **subscription admin** sets the protocol fee. It is capped at 10% by the
contract, but within that range the admin can change it, and the change applies
to mandates that are already open. There is no timelock.

The **vault admin** can repoint `set_subscription` at a different contract. A
malicious or compromised vault admin could point it at a contract that drains
balances.

Any production deployment must put both admin keys behind a multisig. The
testnet deployment does not, because it is a testnet deployment.

## Deliberate design choices

**The vault holds funds, and that is a real tradeoff.** Custody is a liability.
Payflow accepts it because the alternative — charging directly from a user's
wallet — requires either a standing token allowance (the same custody risk with
extra steps) or the subscriber being online at charge time, which defeats the
purpose. Separating custody into its own contract at least means it can be
audited on its own, and subscribers can withdraw at any moment.

**No reentrancy guard.** `debit` calls out to a SEP-41 token. Every code path
writes state *before* transferring, so a hostile token cannot observe a stale
balance. It can still make a charge fail. That is a griefing vector, not a theft
vector.

**Timestamps come from the ledger.** Validators can skew them slightly.
`MIN_PERIOD` is 60 seconds so that skew cannot meaningfully accelerate billing.
