# Simple Minting

```ts
import {
  TxBuilderRunner,
  Value,
  DataConstr,
  Hash28,
  IValuePolicyEntry,
} from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost'
import { getTxBuilder } from './getTxBuilder'
import { user1Details } from './user1'
import { makeValidator } from './makeValidator'

export async function checkRunner() {
  const b = await blockfrost()

  const runner = new TxBuilderRunner(
    await getTxBuilder(b),
    b,
  )

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]

  const alwaysMint = '585401010029800aba2aba1aab9eaab9dab9a4888896600264653001300600198031803800cc0180092225980099b8748000c01cdd500144c9289bae30093008375400516401830060013003375400d149a26cac8009'
  const { validator, hash: policy } = await makeValidator(alwaysMint, [])

  const mintValue: IValuePolicyEntry = {
    policy,
    assets: [{
      name: new Hash28(pkh).toBuffer(),
      quantity: 1n
    }]
  }

  const outputValue = Value.singleAsset(
    policy,
    new Hash28(pkh).toBuffer(),
    1n
  )

  const tx = await runner
    .mintAssets(
      mintValue,
      validator,
      new DataConstr(0, [])
    )
    .addInputs([spendingUtxo])
    .payTo(
      address,
      outputValue
    )
    .setChangeAddress(address)
    .build()

  tx.signWith(XPrv)

  const txHash = await b.submitTx(tx)
  return console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```