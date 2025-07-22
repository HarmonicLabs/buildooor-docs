# OneShot Minting

```ts
import {
  TxBuilderRunner,
  Address,
  Credential,
  DataB,
  DataConstr,
  DataI,
  Hash28,
  UPLCConst,
  Value,
  IValuePolicyEntry,
} from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'

// A OneShot minting policy is a common design pattern on Cardano
// This Validator will only allow you to mint Once (hence `one-shot`)
// The way this is done is by parameterising the validator by a Utxo.OutRef
export async function oneShotMint() {
  const b = await blockfrost()

  // Create TxBuilderRunner instance
  const runner = new TxBuilderRunner(
    await getTxBuilder(b),
    b
  )

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]

  // To use our Utxo Oref as a parameter, we need to apply it in `Constr` format
  // to the validator.
  // Tx Id is a string, so we need to convert it to DataB
  // and the index is a number, so we convert it to DataI
  const oref = new DataConstr(0, [
    new DataB(spendingUtxo.utxoRef.id.toBuffer()),
    new DataI(spendingUtxo.utxoRef.index)
  ])

  const oneShot = '590329010100229800aba2aba1aba0aab9faab9eaab9dab9a4888888966002646465300130053754003370e9000488c8cc00400400c896600200314c0103d87a80008992cc004c010006266e9520003300d0014bd7044cc00c00cc03c0090091806800a0169804801cc02400922222598009802002c4cc8966002600c60186ea8006264944c040c034dd5000c5900b1bae300e300b375400c660066eb0c038c02cdd5001119baf300f300c375400202513259800980080344c8ca6002264b30013008300e3754003133223259800980398089baa0018992cc004c030c048dd5004c4c966002601a60266ea800626466ebcdd318009bab300530153754600a602a6ea8018dd318009bab3005301537546030602a6ea80088c8cc004004008896600200314bd6f7b63044ca60026eb8c05c0066eacc06000660380049112cc004cdc8a45000038acc004cdc7a441000038800c401501944cc074cdd81ba9003374c0046600c00c00280c8603400280c22c8090cc02cdd6180b180b980b98099baa00a23375e602e60286ea8c05cc050dd500099ba548008cc058dd480125eb8226464b300130190018992cc004cdc3a40026eb4c058006264b3001301b0018992cc004c034dd6980c000c4cdc79bae3017003375c602e0031640586034003164060660066eacc018c058dd51803180b1baa0070048b202830180018b202c330013756602e603060306030603060286ea802c00888c9660026016602a6ea8006297adef6c6089bab30193016375400280a0c8cc00400400c896600200314c0103d87a8000899192cc004cdc8802800c56600266e3c014006266e9520003301b30190024bd7045300103d87a8000405d133004004301d003405c6eb8c05c004c0680050182022375c602a60246ea80062c8080c050c044dd5180a18089baa3001301137540046024601e6ea80048c04cc0500062c8068cc018dd6180898071baa00523375e6024601e6ea800400a601a6ea8012602260240049112cc004c02800a2b30013011375400f0038b20248acc004c01800a2b30013011375400f0038b20248b201e403c3010001300c375400f1640286e1d2002402430083009001300800130033754011149a26cac8009'

  // When building the validator, we need to pass the oref in the params array
  // But in order for the validator UPLC to work with this param, it needs to be a UPLCConst
  // the oref is a `DataConstr`, so we can use `UPLCConst.data` to convert it
  const { validator, hash: policy } = await makeValidator(
    oneShot,
    [UPLCConst.data(oref)]
  )

  const mintValue: IValuePolicyEntry = {
    policy,
    assets: [{
      name: new Uint8Array(),
      quantity: 1n
    }]
  }

  const validatorAddress = new Address({
    network: 'testnet',
    paymentCreds: Credential.script(validator.hash)
  })

  const outDatum = new DataConstr(0, [
    new DataB(new Hash28(pkh).toBuffer()),
    new DataI(0)
  ])

  const tx = await runner
    .mintAssets(
      mintValue,
      validator,
      new DataConstr(0, [])
    )
    .addInputs([spendingUtxo])
    .payTo(
      validatorAddress,
      Value.singleAsset(policy, new Uint8Array(), 1n),
      outDatum
    )
    .setChangeAddress(address)
    .build()

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)

  console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```