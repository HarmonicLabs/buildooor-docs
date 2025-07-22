---
sidebar_position: 5
slug: '/examples/sending-assets'
---

# Sending Assets

In this example transaction, we are going to look at apply simple parameters to validators, we will mint an asset and then we will send that asset to a different address.

The main focus is to look at sending `Value`s other than ADA, but we will also see how to apply a parameter to a validator.

The minting policy we will use is parameterised by a pubKeyHash, an address's `paymentCreds`. This is an easy way of limiting a validator by requiring a specific signer in a transaction.

In other words:

You can only mint IF the transaction is signed by the parameterised pkh.

So we will use our `user1` pubKeyHash as our parameter and sign the transaction with this wallet `XPrv`.

we will start by creating a new file called `myParamMinting.ts` and add our imports.

```ts
import { ByteString, DataConstr, Hash28, ITxBuildMint, TxOut, UPLCConst, Value } from '@harmoniclabs/buildooor'
import { user1Details } from './user1.js'
import { getTxBuilder } from './getTxBuilder.js'
import { blockfrost } from './blockfrost.js'
import { makeValidator } from './makeValidator.js'
```

We can create our mint function `myParamMinting()` and set up our `Provider` and `TxBuilder` as normal, also getting our `user1` wallet details and querying the utxos at our address.

```ts
export async function myParamMinting() {
  const b = await blockfrost()
  const txBuilder = await getTxBuilder(b)

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]
```

This validator requires a prameter, the parameter is our pubKeyHash which is our wallet `paymentCredential`, this is needed because the validator ONLY allows minting IF the transaction is signed by the corresponding private key.

This means that when we apply our `pkh` as a parameter, the resulting `Script` will only mint if the `user1` wallet signature is attached.

We need to conver our `pkh` to a `ByteString` so we can then make a `UPLCConst` wich we can apply as the parameter.

```ts
  // We have our basic transaction set up above, with blockfrost initialised and
  // our wallet details loaded,
  // We can now build our asset minting policy
  // We will use a parameterised validator, which will take a public key as a parameter
  // and use it to mint some assets
  const paramMint = '5897010100229800aba2aba1aab9faab9eaab9dab9a9bae002488888896600264646644b30013370e900018039baa00189991198008009bac300c300d300d300d300d300d300d300d300d300a3754601800c6eb8c028c020dd5000912cc00400629422b30013371e6eb8c03000401e2946266004004601a002804100b459006180400098041804800980400098021baa0088a4d13656400801'

  // the PKH is represented as a ByteString in UPLC, so we convert it to a ByteString
  // and then we can turn it into a UPLCConst.bs to apply to the validator
  const pkByteString = new ByteString(new Hash28(pkh).toBuffer())
  const pkParam = UPLCConst.byteString(pkByteString)
```

To apply the parameter to the validator, we use our `makeValidator()` function, passing the compiled code and adding our `pkParam` to the params array.

```ts
  // We can now build our asset minting policy
  // We will use a parameterised validator, which will take a public key as a parameter
  // and use it to mint the asset
  const { validator, hash } = await makeValidator(
    paramMint,
    [pkParam]
  )

```

Now we have our parameterised validator `Script` and the `hash` of that validator, with this info we can make our mint `Value` just as before in [Simple Minting](/examples/SimpleMinting).

our mint `Value` this time has an empty asset name, which we use an empty `Uint8Array()`, and this time we will mint `100` tokens.

Just as in our previous validator, the redeemer is ignored, so we add the same empty `DataConstr` like before, and attach our parameterised `validator`.

```ts
  // We can now make our mint Value from the hash (policyId) and an empty AssetName
  // We will mint 100 of these tokens
  const mintValue = Value.singleAsset(hash, new Uint8Array(), 100)

  // Next we can build our ITxBuildMint object for our transaction
  const mint: ITxBuildMint = {
    value: mintValue,
    script: {
      inline: validator,
      redeemer: new DataConstr(0, [])
    }
  }

```

We also need to create our `output` utxo and calculate the `minUtxoVale` for, just as we did before, with the `getMinimumOutputLovelaces()` method.

```ts

  // We also need to make sure the assets are sent to our wallet
  // so we will set a txOut to our address
  const assetOut = new TxOut({
    address: address,
    value: mintValue,
  })

  // Dont forget to add the minUTxO Value as well
  const minLovelace = txBuilder.getMinimumOutputLovelaces(assetOut)

```

Next create the final output for our transaction, combining the `minLovelaces` with our `mintValue` using `Value.add()`

```ts

  // add the min lovelace to the output value
  const output = new TxOut({
    ...assetOut,
    value: Value.add(assetOut.value, Value.lovelaces(minLovelace))
  })
```

Now we can add all of these elements to our `buildSync` transaction, but remember we also need to specify the `requiredSigners` field by adding our wallet `pkh` to the array.

```ts
  // We can now build our transaction
  // Our validator is parameterised by our PKH and requires that signature to be present
  // so we also manually specify the signer to include in the transaction 
  // ( we still need to sign it with our XPrv )
  const tx = txBuilder.buildSync({
    inputs: [spendingUtxo],
    mints: [mint],
    outputs: [output],
    requiredSigners: [pkh],
    changeAddress: address,
  })

```

Finally we can sign and submit the transaction as normal with the `signWith()` method and `submitTx()`

```ts
  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  console.log(`Transaction submitted: ${txHash}`)
}
```

The above module mints the assets, now we can send these assets to other addresses from `user1`. We are going to create several outputs sending the assets in groups of 20, so there will be 5 outputs instead of 1.

We will start by creating a new file called `sendingAssets.ts` and adding our imports.

```ts
import { Address, ByteString, Hash28, ITxBuildInput, ITxBuildOutput, TxOut, UPLCConst, Value } from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'
```

We define a function called `sendAssets()` and initialise our `Provider`, `TxBuilder` and get `user1` wallet details as normal.

```ts
export async function sendAssets() {
  // In this transaction we are going to look at UTxO selection as well as
  // managing Inputs and Outputs in a more granular way
  // We are going to take the 100 assets we minted, and we are going to send
  // them to ourselves in batches of 20 assets,
  // this means that our transaction will have 5 outputs, each with 20 assets
  const b = await blockfrost()
  const txBuilder = await getTxBuilder(b)

  const { address, pkh, XPrv } = await user1Details()

  ...
}
```

Before we can select our assetUtxos we need to recompile the validator. We go through the same process as before, using the compiled code and applying `user1`'s pkh as a `UPLCConst.byteString()`

```ts
export async function simpleTransaction() {
  ...

  // Before we go an further we need to create our minting policy again.
  // We will need to take the original script and re-parameterise it
  // With our PKH
  const paramMint = '5897010100229800aba2aba1aab9faab9eaab9dab9a9bae002488888896600264646644b30013370e900018039baa00189991198008009bac300c300d300d300d300d300d300d300d300d300a3754601800c6eb8c028c020dd5000912cc00400629422b30013371e6eb8c03000401e2946266004004601a002804100b459006180400098041804800980400098021baa0088a4d13656400801'
  const pkByteString = new ByteString(new Hash28(pkh).toBuffer())
  const pkParam = UPLCConst.byteString(pkByteString)

  ...
}
```

then we can use our `makeValidator()` function to apply the parameter and output the `Script` and the `hash`

```ts
export async function simpleTransaction() {
  ...

  const { validator, hash } = await makeValidator(
    paramMint,
    [pkParam]
  )

  ...
}
```

In order to send assets, we need to include them in the inputs of our transaction, so we need to query to utxos at `user1` address, and filter them for the asset we are looking to spend.

We could just add all of the address utxos, but it would mean that all of the other values stored at this address will be combined into a single change utxo, which can create an unspendable utxo and is generally bad practice - if we are sending one asset, we dont want to include anything else unnecessarily.

We will filter our utxos with the `value.get()` method, this will return an integer representing the amount of the given asset in that utxo, so if the amount is greater than `0` we will return `true` otherwise we will return `false` and it wont be included.

`value.get()` requires the asset we are filtering for, so we add our `policy` (hash) and the empty `Uint8Array()` asset name.

```ts
export async function simpleTransaction() {
  ...

  // We will first get our UTxOs from our wallet address
  const utxos = await b.getUtxos(address)

  // we can then filter our wallet utxos for this asset so we can spend them
  const assetUtxos = utxos.filter(utxo => {
    const value = utxo.resolved.value
    if (value.get(hash, new Uint8Array()) > 0) {
      return true
    }
    return false
  })

  ...
}
```

Now we have a list of utxos with our asset (there should only be one utxo in this list).

We also want to include a utxo with just ADA to cover the minUtxo value of our outputs (as there will be more outputs than inputs we need to include more ADA)

```ts
export async function simpleTransaction() {
  ...

  // We also need to include at least one more UTxO with some ADA to cover
  // The min UTxO Value required by the protocol
  const adaUtxos = utxos.filter(utxo => {
    const value = utxo.resolved.value
    return Value.isAdaOnly(value)
  })

  ...
}
```

because we dont really know how much ADA is in any of these values OR what the exact requirements of the minUtxoValue are, we will add them all to our `ITxBuildInput[]` array. This is not ideal, but for the purposes of simplifying this example we will just add them all.

```ts
export async function simpleTransaction() {
  ...

  // We need to create an ITxBuildInput for the Utxo(s) that we are spending
  // but as it is a wallet UTxO we dont need any info other than the UTxO itself
  const assetInputs: ITxBuildInput[] = assetUtxos.map(utxo => ({
    utxo: utxo,
  }))
    // we concat this with the adaUtxos array, which we map into an `ITxBuildInput[]`
    .concat(adaUtxos.map(utxo => ({ utxo: utxo })))

  ...
}
```

Now we have our inputs, we can build our outputs. We want to create 5 outputs of 20 tokens. We do this by creating a 5 element `Array`, making a `singleAsset()` value of our token with a quantity of `20` and returning a `TxOut` with our wallet address and this `singleAsset` value.

```ts
export async function simpleTransaction() {
  ...

  // Finally we can build our outputs,
  // as we are going to send 20 assets to ourselves in 5 outputs
  // we can easily map this and then apply the minUtxo value
  const outsArray = Array.from({ length: 5 }, (_, i) => {
    const value = Value.singleAsset(hash, new Uint8Array(), 20)
    return new TxOut({
      address: address,
      value: value,
    })
  })

  ...
}
```

now we can calculate and add the `minUtxoValue` for each of these outputs, using `Value.add()` and the `getMinimumOutputLovelaces()` method we have used in the previous examples.

```ts
export async function simpleTransaction() {
  ...

  const outputs = outsArray.map(out => ({
    address: out.address,
    value: Value.add(out.value, Value.lovelaces(txBuilder.getMinimumOutputLovelaces(out)))
  }))

  ...
}
```

As we are sending to and from our `user1` wallet, we dont need anything else to add to our `buildSync` transaction, so we can just add these in along with our `changeAddress` as normal.

```ts
export async function simpleTransaction() {
  ...

  // Now that is complete we can build our transaction with BuildSync
  const tx = txBuilder.buildSync({
    inputs: assetInputs,
    outputs: outputs,
    changeAddress: address,
  })

  ...
}
```

then we can sign the transactions and submit it.

```ts
export async function simpleTransaction() {
  ...

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  return console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```

Return to [Getting Started](/docs/getting-started) to continue ...