# Sending Assets

```ts
import {
  TxBuilderRunner,
  ByteString,
  Hash28,
  UPLCConst,
  Value,
} from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost.js'
import { getTxBuilder } from './getTxBuilder.js'
import { user1Details } from './user1.js'
import { makeValidator } from './makeValidator.js'

export async function simpleTransaction() {
  const b = await blockfrost()

  const runner = new TxBuilderRunner(
    await getTxBuilder(b),
    b
  )

  const { address, pkh, XPrv } = await user1Details()

  const paramMint = '5897010100229800aba2aba1aab9faab9eaab9dab9a9bae002488888896600264646644b30013370e900018039baa00189991198008009bac300c300d300d300d300d300d300d300d300d300a3754601800c6eb8c028c020dd5000912cc00400629422b30013371e6eb8c03000401e2946266004004601a002804100b459006180400098041804800980400098021baa0088a4d13656400801'
  const pkByteString = new ByteString(new Hash28(pkh).toBuffer())
  const pkParam = UPLCConst.byteString(pkByteString)

  const { validator, hash: policy } = await makeValidator(
    paramMint,
    [pkParam]
  )

  const utxos = await b.getUtxos(address)
  const assetUtxos = utxos.filter(utxo =>
    utxo.resolved.value.get(policy, new Uint8Array()) > 0
  )
  const adaUtxos = utxos.filter(utxo =>
    Value.isAdaOnly(utxo.resolved.value)
  )

  // Create 5 outputs of 20 tokens each
  const tx = await runner
    .addInputs([...assetUtxos, ...adaUtxos])
    .setChangeAddress(address)

  // Add 5 outputs with 20 tokens each
  for (let i = 0; i < 5; i++) {
    tx.payTo(
      address,
      Value.singleAsset(policy, new Uint8Array(), 20n)
    )
  }

  const builtTx = await tx.build()

  builtTx.signWith(XPrv)

  const txHash = await b.submitTx(builtTx)

  console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```