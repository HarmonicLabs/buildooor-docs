---
sidebar_position: 6
slug: '/examples/simple-minting'
---

# Simple Minting

Now we have had a look at compiling validators, we can start using them in our transactions.

We will start with an un-parameterised validator, for simplicities sake, and we can take a look at parameters in a different example.

First we will create a file called `simpleMinting.ts` and add our imports

```ts
import { Address, DataConstr, Hash28, ITxBuildInput, ITxBuildMint, ITxBuildOutput, TxOut, Value } from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'
```

then we can create our `simpleMint()` function and initialise our `provider`, `txBuilder` and get our `user1` wallet details

```ts
export async function simpleMint() {
  const b = await blockfrost()
  const txBuilder = await getTxBuilder(b)

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]

  ...
}
```

We need to work with a minting policy in this example, this is a validator we have pre-built. add this string and use it as a parameter for the `makeValidator()` function we created earlier.

`alwaysMint` has no parameters, so we add an empty list where the `UPLCConst[]` parameters would go.

```ts
export async function simpleMint() {
  ...

  const alwaysMint = '585401010029800aba2aba1aab9eaab9dab9a4888896600264653001300600198031803800cc0180092225980099b8748000c01cdd500144c9289bae30093008375400516401830060013003375400d149a26cac8009'

  // We pass an empty list because this validator has no params
  const { validator, hash: policy } = await makeValidator(alwaysMint, [])

  ...
}
```

Now we can access the `Script` which we have called `validator` and the `hash` which we have labelled as `policy` (because the policy Id of minting validator is the script hash)

In order to mint an asset we need to create a `Value` from it, so we can add it to the appropriate fileds (`mint` and `outputs`) in our `buildSync` transaction.

A minted `Value` has no `lovelace` value, so we will use the `singleAsset()` method which creates a `Value` from a single asset.

The parameters for `singleAsset()` are the `policy`, `assetName` and `quantity` in that order.

The `policy` is the hash of the minting validator, which will be a `Hash28` type
The `assetName` is a Uint8Array, which we will make by taking `user1` pubKeyHash. This will need to be converted from a string,
`pkh` is also a `Hash28` so we will create a new `Hash28`, convert it to a `buffer` and then into a `Uint8Array`.
We will only mint a single asset with this transaction, so the `quantity` will be `1`.

```ts
export async function simpleMint() {
  ...

  // for our mint value, we create a single asset,
  // the asset has the policy of the validator.hash
  // and for this example, we use your wallet's pkh as the asset name
  const assetToMint = Value.singleAsset(
    policy,
    new Uint8Array(new Hash28(pkh).toBuffer()),
    1
  )

  ...
}
```

Now we have our asset `Value` we can create our `mint` field for the `buildSync` transaction.

The mint field type is `ITxBuildMint`, which is an object that contains the `value` to mint, and the `script` object.
The `script` object needs to include the validator, either as `inline` or a `refScript`, we will add it `inline` in this example.
The validator ignores the redeemer field, but we still need to attach something. unless redeemers are a sinlge element (such as an integer) they are an object with some fields, so we need to attach it in the `DataConstr` format. we will use index `0` and empty fields because we dont actually need to pass anything in the redeemer.

```ts
export async function simpleMint() {
  ...

  // to mint assets, we need to create an ITxBuildMint object
  // this will include the value to mint, the script, and the minting Redeemer
  const mint: ITxBuildMint = {
    value: assetToMint,
    script: {
      inline: validator,
      redeemer: new DataConstr(0, [])
    }
  }

  ...
}
```

Every transaction also needs at least one `input` to pay for the transaction fees, so we will attach our first utxo returned from querying our wallet address utxos, just like before. This needs to be added as an `ITxBuildInput`

```ts
export async function simpleMint() {
  ...

  // remember we are using a wallet utxo so we dont need any script info here
  const input: ITxBuildInput = {
    utxo: spendingUtxo
  }

  ...
}
```

When creating the output, we cant just add the minted value, Cardano requires a minimum amount of lovelace per transaction as a sibyl resistance mechanism. This is called a `minUtxoValue` and we need to calculate this in order to include ONLY what is needed for the transaction, not any more.

We know what we want to send in this output, the minted asset, but we dont yet know what the `minUtxoValue` is, so we will create our output first as a `TxOut`, with our `user1` wallet address and the minted value.

Then we can calculate the `minUtxoValue` with the `getMinimumOutputLovelaces()` method in `TxBuilder`.

We need to pass our `TxOut` to this function, which will return an integer representing the `minUtxoValue` in `lovelace`

```ts
export async function simpleMint() {
  ...

  // Because we are only sending an asset, we need to calculate the minUtxoValue
  // we build the output as TxOut first, then we can calculate the minimum lovelace
  // required for the output to be valid
  const rawOutput = new TxOut({
    address: Address.fromBech32(address),
    value: assetToMint
  })

  // calculate the min lovelace Value
  const minLovelace = txBuilder.getMinimumOutputLovelaces(rawOutput)

  ...
}
```

Now we need to add this `minUtxoValue` to our `TxOut`, we can consolidate these steps into a single function, but I have separated them out to make things simpler to demonstrate.

We will create a new `output` variable which will be  a new `TxOut` with the previous `rawOutput` but adding our `minLovelaceValue` that we just calculated.

```ts
export async function simpleMint() {
  ...

  // add the min lovelace to the output value
  const output = new TxOut({
    ...rawOutput,
    value: Value.add(rawOutput.value, Value.lovelaces(minLovelace))
  })

  ...
}
```

The last step is to add the `mint`, `input` and `output` to `buildSync` so we can contruct the transaction ready to sign,

we also include our `changeAddress` which will recieve any leftover ADA from our `input`.

```ts
export async function simpleMint() {
  ...

  // we need to include our mint in the transaction
  const tx = txBuilder.buildSync({
    inputs: [input],
    mints: [mint],
    outputs: [output],
    changeAddress: address,
  })

  ...
}
```

Finally, we can sign and submit the transaction as before, with our wallet `XPrv` and our `Provider`

```ts
export async function simpleMint() {
  ...

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  return console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```

Return to [Getting Started](/docs/getting-started) to continue ...