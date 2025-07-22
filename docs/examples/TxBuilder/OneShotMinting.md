---
sidebar_position: 3
slug: '/examples/one-shot-minting'
---

# One Shot Minting

In this example we will take a look at a common design pattern in Cardano, the `One Shot` minting policy. This is often used as a way to create a unique token that can be referenced as a source of truth. This is because a one shot minting policy is designed so that it can only be used once.

This is done by parameterising a validator by an output reference, and the validator only mints if a transaction input has a matching output reference.

Because output references are unique to utxos, that means there will only ever be one utxo with a matching output reference, so once it is consumed in the minting transaction, that transaction, and thus the validator, can never be used to mint again.

In this example we will look at a more complex form of validator parameter, the output reference, which has 2 elements, the transaction id and the transaction index.

An output reference must be applied as a `UPLCConst` in the `data` format, this means we need to create a `DataConstr` of that output reference to apply it as a parameter.

## Minting Transaction

We will start by making a new file called `oneShotMinting.ts` and add our imports

```ts
import { Address, Credential, DataB, DataConstr, DataI, Hash28, ITxBuildInput, ITxBuildMint, ITxBuildOutput, TxOut, UPLCConst, Value } from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'
```

We can then define our `oneShotMint()` function and initialise our `Provider`, `TxBuilder` and our `user1` wallet.

```ts
// A OneShot minting policy is a common design pattern on Cardano
// This Validator will only allow you to mint Once (hence `one-shot`)
// The way this is done is by parameterising the validator by a Utxo.OutRef
export async function oneShotMint() {
  const b = await blockfrost()
  const txBuilder = await getTxBuilder(b)

  const { address, pkh, XPrv } = await user1Details()

  ...
}
```

Then we can query our address utxos, and select index 0 as our spending Utxo, which we will use as our one-shot parameter.

```ts

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]

```

To parameterise the validator, we need to apply the output reference of this utxo in the `DataConstr` format

```ts

  // To use our Utxo Oref as a parameter, we need to apply it in `Constr` format
  // to the validator.
  // Tx Id is a string, so we need to convert it to DataB
  // and the index is a number, so we convert it to DataI
  const oref = new DataConstr(0, [
    new DataB(spendingUtxo.utxoRef.id.toBuffer()),
    new DataI(spendingUtxo.utxoRef.index)
  ])

```

Then we can apply it to our oneshot script as we have done before. 

To accept a `DataConstr` as a parameter, we need to apply it as a `UPLCConst.data()`

```ts

  const alwaysMint = '590329010100229800aba2aba1aba0aab9faab9eaab9dab9a4888888966002646465300130053754003370e9000488c8cc00400400c896600200314c0103d87a80008992cc004c010006266e9520003300d0014bd7044cc00c00cc03c0090091806800a0169804801cc02400922222598009802002c4cc8966002600c60186ea8006264944c040c034dd5000c5900b1bae300e300b375400c660066eb0c038c02cdd5001119baf300f300c375400202513259800980080344c8ca6002264b30013008300e3754003133223259800980398089baa0018992cc004c030c048dd5004c4c966002601a60266ea800626466ebcdd318009bab300530153754600a602a6ea8018dd318009bab3005301537546030602a6ea80088c8cc004004008896600200314bd6f7b63044ca60026eb8c05c0066eacc06000660380049112cc004cdc8a45000038acc004cdc7a441000038800c401501944cc074cdd81ba9003374c0046600c00c00280c8603400280c22c8090cc02cdd6180b180b980b98099baa00a23375e602e60286ea8c05cc050dd500099ba548008cc058dd480125eb8226464b300130190018992cc004cdc3a40026eb4c058006264b3001301b0018992cc004c034dd6980c000c4cdc79bae3017003375c602e0031640586034003164060660066eacc018c058dd51803180b1baa0070048b202830180018b202c330013756602e603060306030603060286ea802c00888c9660026016602a6ea8006297adef6c6089bab30193016375400280a0c8cc00400400c896600200314c0103d87a8000899192cc004cdc8802800c56600266e3c014006266e9520003301b30190024bd7045300103d87a8000405d133004004301d003405c6eb8c05c004c0680050182022375c602a60246ea80062c8080c050c044dd5180a18089baa3001301137540046024601e6ea80048c04cc0500062c8068cc018dd6180898071baa00523375e6024601e6ea800400a601a6ea8012602260240049112cc004c02800a2b30013011375400f0038b20248acc004c01800a2b30013011375400f0038b20248b201e403c3010001300c375400f1640286e1d2002402430083009001300800130033754011149a26cac8009'
  // When building the validator, we need to pass the oref in the params array
  // But in order for the validator UPLC to work with this param, it needs to be a UPLCConst
  // the oref is a `DataConstr`, so we can use `UPLCConst.data` to convert it
  const { validator, hash: policy } = await makeValidator(
    alwaysMint, 
    [UPLCConst.data(oref)]
  )

```

Now we have our validator details, we can continue as normal with setting up the mint, by creating our mint `Value` with the `singleAsset()` method and create the `ITxBuildMint` value that `buildSync` expects.

Again, we ignore the redeemer in this validator, so we will add an empty `DataConstr` in the redeemer field.

```ts
  
  // for our mint value, we create a single asset,
  // the asset has the policy of the validator.hash
  // and for this example we will have an empty AssetName
  const assetToMint = Value.singleAsset(
    policy,
    new Uint8Array(),
    1
  )

  // to mint assets, we need to create an ITxBuildMint object
  // this will include the value to mint, the script, and the minting Redeemer
  const mint: ITxBuildMint = {
    value: assetToMint,
    script: {
      inline: validator,
      redeemer: new DataConstr(0, [])
    }
  }

```

We need to add our parameterised utxo as an input or validatoin will fail, so we create an `ITxBuildInput` from the `spendingUtxo`

```ts

  // We need to spend the param Utxo in the transaction, so we make sure to include it
  // in the inputs ( or we wont be able to mint )
  const input: ITxBuildInput = {
    utxo: spendingUtxo
  }

```

OneShot policies serve a specific purpose in a dapp, so generally they are sent to the validator address, or an unspendable script address, not the users wallet.

In this example we will lock it in the script address, which we will need to create using the `Address` class.

We use the validator `hash` as the payment credential for calculating our `validatorAddress` (which is the same as the policyId of our asset) using the `Credential.script()` method to create the `paymentCreds` of the address.

There is not staking element needed for this validator address, so we can just ignore that field.

```ts

  // Oneshots are generally used for creating a reference point for other Validators,
  // so we dont want to send it to our wallets, instead we lock it into the validator.
  // Our one-shot validator has a spending purpose, so we can use the address of the validator
  // as the output address.
  const validatorAddress = new Address({
    network: 'testnet',
    paymentCreds: Credential.script(validator.hash)
  });

```

Although this example doesnt need a datum, a one-shot generally stores some data with it, and we havent looked at creating datums yet in our examples, so we will add one here.

Datums, like redeemers, dont NEED to be in the `DataConstr` format, we could have a single integer using `DataI()` or something similar. But most of the time, a datum will have more than one piece of data in it, and so will need to be represented in the `DataConstr` format.

In our example, we will add our `pkh` and a `0` integer.

```ts

  // our validator output will also get a datum, stored at that uxto, 
  // this will also need to be a DataConstr
  // We will add our wallet PKH and an integer 0 to the datum which we can increment later
  const outDatum = new DataConstr(0, [
    new DataB(new Hash28(pkh).toBuffer()),
    new DataI(0)
  ])

```

We can then add this along with our minLovelace value and our oneshot token value to a `TxOut`

```ts

  const rawOutput = new TxOut({ 
    address: validatorAddress,
    value: assetToMint,
    // our validator output will also get a datum, stored at that uxto, 
    // this will also need to be a DataConstr
    datum: outDatum
  })

  // calculate the min lovelace Value
  const minLovelace = txBuilder.getMinimumOutputLovelaces(rawOutput)

  // add the min lovelace to the output value
  const output = new TxOut({
    ...rawOutput,
    value: Value.add(rawOutput.value, Value.lovelaces(minLovelace))
  })

```

Now that we have set our `mint`, `input` and `output` we can construct the transaction using `buildSync` and add our `changeAddress` for any leftover `Value` from our input.

```ts

  // we need to include our mint in the transaction
  const tx = txBuilder.buildSync({
    inputs: [input],
    mints: [mint],
    outputs: [output],
    changeAddress: address,
  })

```

Finally we can sign and submit the transaction as normal.

But before we do that, we need to actually save either the validator after it has been parameterised - OR - the oref variable so we can rebuild the validator.

This is important because otherwise we could lose the initial parameter and we wont be able to rebuild the script to update our oneshot token datum or burn the asset in the future.

Even if we didnt want to spend it or burn it, we still need to be able to find the asset, so we will likely need to rebuild the validator in the future even if we are just going to be using the utxo as a reference input.

We will save our parameters to a simple json file to keep things simple, and human readable, using `fs.writeFileSync`

```ts

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  const oneshotParam = {
    id: spendingUtxo.utxoRef.id,
    index: spendingUtxo.utxoRef.index
  }

  fs.writeFileSync('oneshotParam.json', JSON.stringify(oneshotParam, null, 2))

  return console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```

Now we can run this transaction and we will save the parameter UTxO details so we can re-use them in our next transaction, when we update the Datum.

## Spending Transaction

For our spending transaction, the goal is to increment the Integer at the datum of our oneShot Token.

To do this we need to spend the asset UTxO and send it back to the validator address with the updated Datum.

We will start by creating a file callet `oneShotSpend.ts` and we will set up our usual imports.

```ts
import { Address, Credential, DataB, DataConstr, DataI, Hash28, ITxBuildInput, ITxBuildMint, ITxBuildOutput, TxOut, UPLCConst, Value } from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'
```

Then as normal we will create a function and initialise our `blockfrostProvider`, txBuilder & get our wallet utxos.

```ts
// A OneShot minting policy is a common design pattern on Cardano
// This Validator will only allow you to mint Once (hence `one-shot`)
// The way this is done is by parameterising the validator by a Utxo.OutRef
export async function oneShotSpend() {
  const b = await blockfrost()
  const txBuilder = await getTxBuilder(b)

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]
```

We need to pull our parameter details so we will read the information from the file we created in our mint transaction, then rebuild our parameter to apply to the validator just like we did previously

```ts
  const params = JSON.parse(fs.readFileSync('oneshotParam.json', 'utf8'))
  
  const oref = new DataConstr(0, [
    new DataB(params.id.toBuffer()),
    new DataI(params.index)
  ])

  const alwaysMint = '590329010100229800aba2aba1aba0aab9faab9eaab9dab9a4888888966002646465300130053754003370e9000488c8cc00400400c896600200314c0103d87a80008992cc004c010006266e9520003300d0014bd7044cc00c00cc03c0090091806800a0169804801cc02400922222598009802002c4cc8966002600c60186ea8006264944c040c034dd5000c5900b1bae300e300b375400c660066eb0c038c02cdd5001119baf300f300c375400202513259800980080344c8ca6002264b30013008300e3754003133223259800980398089baa0018992cc004c030c048dd5004c4c966002601a60266ea800626466ebcdd318009bab300530153754600a602a6ea8018dd318009bab3005301537546030602a6ea80088c8cc004004008896600200314bd6f7b63044ca60026eb8c05c0066eacc06000660380049112cc004cdc8a45000038acc004cdc7a441000038800c401501944cc074cdd81ba9003374c0046600c00c00280c8603400280c22c8090cc02cdd6180b180b980b98099baa00a23375e602e60286ea8c05cc050dd500099ba548008cc058dd480125eb8226464b300130190018992cc004cdc3a40026eb4c058006264b3001301b0018992cc004c034dd6980c000c4cdc79bae3017003375c602e0031640586034003164060660066eacc018c058dd51803180b1baa0070048b202830180018b202c330013756602e603060306030603060286ea802c00888c9660026016602a6ea8006297adef6c6089bab30193016375400280a0c8cc00400400c896600200314c0103d87a8000899192cc004cdc8802800c56600266e3c014006266e9520003301b30190024bd7045300103d87a8000405d133004004301d003405c6eb8c05c004c0680050182022375c602a60246ea80062c8080c050c044dd5180a18089baa3001301137540046024601e6ea80048c04cc0500062c8068cc018dd6180898071baa00523375e6024601e6ea800400a601a6ea8012602260240049112cc004c02800a2b30013011375400f0038b20248acc004c01800a2b30013011375400f0038b20248b201e403c3010001300c375400f1640286e1d2002402430083009001300800130033754011149a26cac8009'

  // When building the validator, we need to pass the oref in the params array
  // But in order for the validator UPLC to work with this param, it needs to be a UPLCConst
  // the oref is a `DataConstr`, so we can use `UPLCConst.data` to convert it
  const { validator, hash: policy } = await makeValidator(
    alwaysMint,
    [UPLCConst.data(oref)]
  )
```

Now we can make our asset `Value` using the validator `hash`

```ts
  // for our oneShot value, we create a single asset,
  // the asset has the policy of the validator.hash
  // and for this example we will have an empty AssetName
  const assetToMint = Value.singleAsset(
    policy,
    new Uint8Array(),
    1
  )
```

We make our wallet input to cover the transaction fee, remember it needs to be an `ITxBuildInput`

```ts
  const walletInput: ITxBuildInput = {
    utxo: spendingUtxo
  }
```

We also need to get the UTxO with the oneshot token.

To do this we will make our validator address, get the UTxOs at that address, and because the validator could hold other UTxOs, we will filter them for our oneShot Token.

```ts
  const validatorAddress = new Address({
    network: 'testnet',
    paymentCreds: Credential.script(validator.hash)
  });

  const validatorUtxos = await b.getUtxos(validatorAddress)
  const oneShotUtxo = validatorUtxos.find(utxo => utxo.resolved.value.get(policy, new Uint8Array()) > 0)

  if (!oneShotUtxo) {
    throw new Error('No one-shot UTxO found')
  }
```

To build our oneShotInput we need to attach the validator information that allow us to spend.

This is an `inputScript` where the `script` field is the validator, and the `redeemer` field will be an empty `DataConstr` with an index of `0`

```ts
  const oneShotIn: ITxBuildInput = {
    utxo: oneShotUtxo,
    inputScript: {
      script: validator,
      redeemer: new DataConstr(0, [])
    }
  }
```

Whenever we are spending UTxOs from a validator we will need to attach the relevant script & the redeemer according to the specification set in the validator.

If there is a mismatch in this info, the transaction will not go through.

We can build our new datum and update the Integer field to `1` as we are incrementing it from `0` which we set during minting

```ts
  // our validator output will also get a datum, stored at that uxto, 
  // this will be the same as the input datum from our mint transaction,
  // except we will increment the Integer
  const outDatum = new DataConstr(0, [
    new DataB(new Hash28(pkh).toBuffer()),
    new DataI(1)
  ])
```

We have all of the information we need to build our new transaction output for our `oneShotToken`.

Just as before we will build the output, then calculate our minUtxo value and add it to our `TxOut`

```ts
  const rawOutput = new TxOut({
    address: validatorAddress,
    value: assetToMint,
    datum: outDatum
  })

  const minLovelace = txBuilder.getMinimumOutputLovelaces(rawOutput)

  const output = new TxOut({
    ...rawOutput,
    value: Value.add(rawOutput.value, Value.lovelaces(minLovelace))
  })
```

Now we can sign and submit the transaction. We dont need to save the paramters this time as we already have them saved.

```ts
  const tx = txBuilder.buildSync({
    inputs: [walletInput, oneShotIn],
    outputs: [output],
    changeAddress: address,
  })

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  return console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```

Return to [Getting Started](/docs/getting-started) to continue ...