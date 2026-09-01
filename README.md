# Payflow

Payflow is recurring payments for the Stellar network.

## The problem

Stellar payments are push-only. The account that holds the funds must sign every
transfer. That works for a one-off payment and breaks completely for a
subscription: a merchant cannot bill a customer every month unless the customer
signs a transaction every month, and no customer does that.

So every subscription business on Stellar solves it the same two ways. Either
they hold customer funds off-chain, or they ask the customer for a key with
spending power. Both put the merchant back in the position of trust that using a
blockchain was supposed to remove.

This is not a small corner of commerce. Subscriptions are the default billing
model for software, media, telecoms, insurance premiums, gym memberships, and
utilities. Card networks handle it with a "merchant-initiated transaction" — a
stored mandate the merchant charges against without the cardholder present.
Stellar has no equivalent primitive.

## What Payflow adds

A **mandate**: a standing authorization a subscriber signs once, bounded on
every axis that matters.

| Bound | Meaning |
|---|---|
| Amount | Fixed at subscribe time. The merchant cannot raise it later. |
| Period | Fixed. At most one charge per period. |
| Count | Optional hard cap on total charges. |
| Revocation | The subscriber can cancel unilaterally, at any time. |
| Funding | Charges come from a vault the subscriber can drain at will. |

Once a mandate exists, **anyone** can trigger settlement when it falls due. The
mandate is the authorization, so it does not matter who submits the transaction
— the merchant, an automated keeper, or a stranger. Nobody can take more than
the mandate permits, and no operator has to stay online for billing to work.

## How it works

1. **The merchant publishes a plan.** Price, billing period, and token go into
   the plan registry. It is public and immutable except for an on/off switch.

2. **The subscriber funds a vault and opens a mandate.** One signature. The
   plan's terms are copied into the mandate at this moment and never re-read.

3. **Anyone settles the charge when it comes due.** The subscription contract
   checks the schedule, debits the vault, pays the merchant, and takes the
   protocol fee.

4. **The subscriber leaves whenever they want.** Cancel is immediate.
   Withdrawing from the vault starves the mandate. Nothing is ever locked.

## Status

Deployed to **Stellar testnet**. Unaudited. Do not use with real funds.

| Contract | ID |
|---|---|
| Plan registry | `CABL5QWEQKMAIDXTQCHEJ2TAAPYUHSQDY73Z4LIK6BOBGYUYO7RG6HJA` |
| Vault | `CBWDD6KVMLEH2D5IVPHZJW4TLLL2RVWGNJRN2US6INNBFTH35K2KYSF7` |
| Subscription | `CBKQVZGZJMY47JFRCOF2RFKEMTTYKUF4LI54AOB6EGDFODNZU7MSUW6K` |
