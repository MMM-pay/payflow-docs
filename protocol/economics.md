---
title: Economics
nav_order: 3
---

# Economics

## The fee

Payflow takes a protocol fee on each settled charge, expressed in **basis
points** (hundredths of a percent). There is no floating point arithmetic
anywhere in the contracts.

```
fee             = amount × fee_bps ÷ 10 000
merchant_amount = amount − fee
```

The fee is capped in the contract at `MAX_FEE_BPS = 1000`, which is 10%. The
admin cannot exceed that ceiling even by mistake.

The testnet deployment runs at **100 bps (1%)**.

## A worked example

A merchant publishes a plan at **10 XLM per month**. Ten subscribers open
mandates. The fee is 100 bps.

Per charge, in stroops (1 XLM = 10 000 000 stroops):

| Line | Stroops | XLM |
|---|---:|---:|
| Charge amount | 100 000 000 | 10.0 |
| Protocol fee (1%) | 1 000 000 | 0.1 |
| Merchant receives | 99 000 000 | 9.9 |

Across ten subscribers for one month:

| Line | XLM |
|---:|---:|
| Gross billed | 100.0 |
| Protocol fee | 1.0 |
| Merchant receives | 99.0 |

Over twelve months, assuming no churn: the merchant receives **1 188 XLM** and
the protocol collects **12 XLM**.

## Real numbers from the testnet deployment

These are actual settled charges, not projections:

| Item | Value |
|---|---|
| Plan price | 10 000 000 stroops (1 XLM) |
| Period | 60 seconds |
| Charges settled | 2 |
| Merchant received per charge | 9 900 000 stroops (0.99 XLM) |
| Fee per charge | 100 000 stroops (0.01 XLM) |
| Vault before | 50 000 000 stroops (5 XLM) |
| Vault after | 30 000 000 stroops (3 XLM) |

## Rounding

Integer division rounds the fee **down**, which favours the merchant.

The practical consequence is a floor: at 100 bps, any charge below 100 stroops
produces a fee of zero. For XLM that is 0.00001 XLM — far below any realistic
subscription price — but a token with fewer decimals could hit it. Merchants
pricing in unusual tokens should check.

## Transaction costs

Someone pays the Stellar transaction fee to settle each charge. Today that is
the keeper. At Stellar's base fee of 100 stroops (0.00001 XLM), settling a
mandate every month for a year costs the keeper roughly **0.00012 XLM**.

This is the economic reason Payflow is a Stellar product rather than a general
one. A protocol that skims 1% of a 10 XLM subscription earns 0.1 XLM per charge
against a settlement cost of 0.00001 XLM — four orders of magnitude of headroom.
On a chain where a transaction costs a dollar, recurring micro-billing simply
does not close.
