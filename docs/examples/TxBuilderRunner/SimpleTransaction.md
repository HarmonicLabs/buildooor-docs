# Simple Transaction

```ts
import {
  TxBuilderRunner,
  Value,
} from '@harmoniclabs/buildooor'
import { blockfrost } from './blockfrost'
import { getTxBuilder } from './getTxBuilder'
import { user1Details } from './user1'

export async function checkRunner() {
  const b = await blockfrost()

  const runner = new TxBuilderRunner(
    await getTxBuilder(b),
    b,
  )

  const { address, pkh, XPrv } = await user1Details()

  const utxos = await b.getUtxos(address)
  const spendingUtxo = utxos[0]
  const adaToSend = 10000000 // 10 ADA in lovelace

  const recipientAddress = 'addr_test1qr8kkwha4wjzvppzsnlhrj9pw4dxe5yv9hsq6eselt9kcyv3w8xrhyu8n7glnq7xdgkddyrdf6c35s8gqr7ph6mnz7fqh6c40y'

  const valueToSend = Value.lovelaces(adaToSend)

  const tx = await runner
    .addInputs([spendingUtxo])
    .payTo(
      recipientAddress,
      valueToSend
    )
    .setChangeAddress(address)
    .build()

  tx.signWith(XPrv)

  console.log(tx)

  const txHash = await b.submitTx(tx)

  console.log(`Transaction submitted successfully with hash: ${txHash}`)
}
```