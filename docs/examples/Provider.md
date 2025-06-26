# Setting Up A Provider

The most familiar way for most people to connect to the chain is with `Blockfrost`.

to do this you will need to use a different library `blockfrost-pluts`, you will also need your Blockfrost API key:

```
npm i @harmoniclabs/blockfrost-pluts
```

once installed you are able to set up your provider

```
import { BlockfrostPluts } from "@harmoniclabs/blockfrost-pluts";

export async function blockfrost() {
    return new BlockfrostPluts({
        projectId: <BLOCKFROST_API_KEY>,
    });
}
```

with this provider set up we are able to connect our dApp to the blockchain, to query utxos, sign and submit transactions.

