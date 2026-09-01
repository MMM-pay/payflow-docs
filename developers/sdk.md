# TypeScript SDK

`src/lib/payflow.ts` in the frontend repo wraps every contract function. It has
no framework dependency and can be lifted into any TypeScript project.

## Reading state

Reads run as a simulation. They cost nothing and submit nothing. The
`sourceAddress` only has to be an account that exists.

```ts
import { payflow } from "@/lib/payflow";

const plan = await payflow.getPlan(address, 1);
// { id: 1n, merchant: "GB34...", amount: 10000000n, period: 60n, active: true }

const balance = await payflow.vaultBalance(address, address, tokenId);
// 30000000n

const due = await payflow.isDue(address, 1);
// true
```

Every numeric field comes back as `bigint`. Never convert it to `number` for
arithmetic on money.

## Writing state

Writes need a signer. `signXdr` takes an unsigned XDR string and returns a
signed one — the wallet context supplies it.

```ts
const { address, signXdr } = useWallet();

const hash = await payflow.deposit(address, signXdr, tokenId, 50_000_000n);
const mandateId = await payflow.subscribe(address, signXdr, planId, 3);
await payflow.cancel(address, signXdr, mandateId);
```

`writeContract` builds the transaction, runs `prepareTransaction` to attach
authorization entries and the resource footprint, hands it to the wallet, sends
it, and polls until the network confirms. Skipping `prepareTransaction` produces
a transaction the network rejects.

## Encoding arguments

ScVal types are explicit. Guessing them is the most common source of silent
failure.

```ts
import { arg } from "@/lib/payflow";

arg.address("GB34...")   // Address
arg.u64(1)               // u64  — plan and mandate ids
arg.u32(3)               // u32  — max_charges, fee_bps
arg.i128(10_000_000n)    // i128 — every amount
arg.bool(true)           // bool
```

## Calling a function with no wrapper

```ts
import { readContract, writeContract, arg } from "@/lib/payflow";

const plans = await readContract<bigint[]>(
  registryId,
  "merchant_plans",
  address,
  [arg.address(merchant)],
);

const hash = await writeContract(
  subscriptionId,
  "charge",
  address,
  signXdr,
  [arg.u64(mandateId)],
);
```

## Errors

Raw failures look like `Error(Contract, #6)`, which means nothing to a user.
`decodeContractError` maps the code — scoped to the right contract — onto a
sentence:

```
#6 on subscribe/charge  ->  "This subscription is not due yet."
#4 on deposit/withdraw  ->  "Not enough balance in your vault."
```

When you add a contract error, add it to the `ERRORS` map too.

## Amounts

Use the helpers in `src/lib/format.ts`. They are `bigint`-based throughout.

```ts
import { fromStroops, toStroops } from "@/lib/format";

fromStroops("9900000")  // "0.99"
toStroops("2.5")        // 25000000n
toStroops("2.55555555") // throws: at most 7 decimal places
```

`toStroops` rejects anything that is not a positive decimal. Do not replace it
with `parseFloat`.
