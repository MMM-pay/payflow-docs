# REST API

Served by `payflow-backend`. Read-only and unauthenticated — everything it
returns is already public on-chain.

All amounts are **strings in the token's smallest unit** (stroops for XLM).
They are never JSON numbers, because JSON numbers are IEEE doubles and would
lose precision on large i128 values.

## Health

```
GET /health
```

```json
{
  "status": "ok",
  "network": "testnet",
  "contracts": {
    "planRegistry": "CDLGJKHOZ4UY4HJXFND7VPEGAFJOBQB7E2W7IGO3JIJLZ6WIX2NCIQ3V",
    "vault": "CABD66DE3FEREIHPS77XUFRTA5EBSE3RZLCRPE3MOYHO42AH7NEAERL5",
    "subscription": "CAM2ISZPOZXMYUIC7WPAXKTU6I4QY4HGWLGATK65EWOPRMTHKAWBMMKR"
  }
}
```

## Plans

```
GET /api/plans?merchant=<G...>&limit=50
GET /api/plans/:id
```

```json
{
  "plans": [
    {
      "id": 1,
      "merchant": "GB343X4K4TIYARSAAQZ55RIZY2YLFNOQCM3LNQSRBV56TMVGIF55WCOH",
      "token": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
      "amount": "10000000",
      "period": 60,
      "active": 1,
      "ledger": 4447697
    }
  ]
}
```

## Mandates

```
GET /api/mandates?subscriber=<G...>&merchant=<G...>&status=Active
GET /api/mandates/:id
GET /api/mandates/:id/charges
```

```json
{
  "mandates": [
    {
      "id": 1,
      "subscriber": "GAIJKXL3Q45ZR32QRDVPUDD4DPLT5GYCHJFESQPHHWJRYX4YZCG25JJR",
      "merchant": "GB343X4K4TIYARSAAQZ55RIZY2YLFNOQCM3LNQSRBV56TMVGIF55WCOH",
      "plan_id": 1,
      "amount": "10000000",
      "period": 60,
      "next_charge": 1788263112,
      "last_charge": 1788263052,
      "charges_made": 2,
      "max_charges": 0,
      "status": "Active"
    }
  ]
}
```

Charge history:

```json
{
  "charges": [
    {
      "tx_hash": "ef182aaeb5d052833912bda4d38b282f1247da9ae5f1666937e811fa2f0d28ea",
      "mandate_id": 1,
      "amount": "9900000",
      "fee": "100000",
      "charges_made": 2,
      "ledger": 4447893
    }
  ]
}
```

`amount` is the merchant's net share. Gross is `amount + fee`.

## Due mandates

```
GET /api/due?limit=50
```

What the keeper works from. Reflects schedule and status; it does **not**
verify the vault can cover the charge.

## Summaries

```
GET /api/merchants/:address/summary
```

```json
{
  "merchant": "GB343X4K4TIYARSAAQZ55RIZY2YLFNOQCM3LNQSRBV56TMVGIF55WCOH",
  "activeMandates": 1,
  "totalCollected": "19800000",
  "chargeCount": 2,
  "mrr": "432000000000"
}
```

`mrr` normalises every active mandate to a 30-day month. A short-period demo
plan therefore yields a very large figure — that is arithmetic, not a bug.

```
GET /api/subscribers/:address/summary
```

`totalPaid` includes the protocol fee, since the subscriber paid it.

## Stats

```
GET /api/stats
```

```json
{
  "plans": 1,
  "mandates": 1,
  "activeMandates": 1,
  "charges": 2,
  "lastIndexedLedger": 4448232
}
```

`lastIndexedLedger` is the freshness signal. If it stops advancing, the indexer
is stuck.

## Errors

| Status | Body | Meaning |
|---|---|---|
| 404 | `{"error":"plan_not_found"}` | No such plan |
| 404 | `{"error":"mandate_not_found"}` | No such mandate |
| 404 | `{"error":"not_found"}` | Unknown route |
| 500 | `{"error":"internal_error"}` | Unhandled failure |

## Treat this as a cache

The API serves a projection rebuilt from chain events. It can be deleted and
rebuilt at any time. **Never treat it as authoritative for money** — read the
contract for that. Frontend clients should degrade gracefully when it is
unreachable, as `src/lib/api.ts` does by returning `null` rather than throwing.
