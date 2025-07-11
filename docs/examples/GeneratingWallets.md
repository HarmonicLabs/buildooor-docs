---
sidebar_position: 2
slug: '/examples/generating-wallets'
---

# Generating Wallets

We are going to look at a simplified version of wallet/key generation, but before we get into it, to learn more about keys and accounts see this more detailed documentation here:

[Account Address Generation](/examples/account-address-generation)

We are going to only look at what we need here, but the above link covers this topic in a lot more detail.

---

Generally wallets are set up using a Wallet application, but in some cases you need something you can program quickly and easily without needing to set up a GUI.

Particularly useful for learning and testing code, Buildooor provides all of the tools we need to generate keys, wallets and addresses so we can focus on transaction building and testing.

We will start by creating a file called `keyFromSeed.ts` which we will use to generate the initial credentials and save them to our environment.

> Note: We also need to install the bip39 package for generating our wallet mnemonic

```ts
import { generateMnemonic, mnemonicToEntropy, validateMnemonic } from 'bip39';
import { XPrv, harden, Address, StakeAddress, NetworkT } from '@harmoniclabs/buildooor';
import fs from 'node:fs';


// We create a function to generate a mnemonic seed phrase
export function genSeedPhrase(): string {
  return generateMnemonic(256);
}

//creates entropy from seed
export const seedPhraseToEntropy = async (seed_phrase: string) => {
  return mnemonicToEntropy(seed_phrase)
}

// generate the root private key using the `fromEntropy()` method
export const genRootPrivateKey = async (entropy: any) => {
    return XPrv.fromEntropy(entropy, '')
}

// derive the specified address from our root private key
export const genAddressPrv = async (xprv_root: XPrv, accIndex: number, addressType: number, addressIndex: number) => {
  return xprv_root
  .derive(harden(1852))
  .derive(harden(1815))
  .derive(harden(accIndex))
  .derive(addressType)
  .derive(addressIndex)
}

// combine these functions together to write our `seed.txt` and `rootKey.txt`
export async function generateWallet() {
  const seed = genSeedPhrase();
  
  // write seed phrase to seed.txt
  fs.writeFileSync('seed.txt', seed);
  console.log("Seed phrase saved to seed.txt");

  const entropy = await seedPhraseToEntropy(seed);
  const rootKey = await genRootPrivateKey(entropy);

  // example call for generating a wallet address
  const accountAddressKeyPrv = await genAddressPrv(rootKey, 0, 0, 0);

  // write root private key to rootKey.txt
  fs.writeFileSync('rootKey.txt', rootKey.toBech32());
  return console.log("Root key saved to rootKey.txt");
}

generateWallet()
```

we will re use some of these functions which is why we export them individually.

Now we have a seed and a root privateKey we can derive addresses and keys as needed for testing our transactions.

---

To allow us to easily work with a single wallet we will create another file that will always return the same address details from our seed phrase, we will call this wallet `user1`.

create a new file called `user1.ts` and we can use our newly created seed phrase and keys functions to generate our wallet dynamically.

```ts
import { Address } from "@harmoniclabs/buildooor";
// import our keyFromSeed functions to generate our wallet details
import { genAddressPrv, genRootPrivateKey, seedPhraseToEntropy } from "./keyFromSeed.js";

export async function user1Details() {
    const seed = <import your seed.txt here>
    const entropy = await seedPhraseToEntropy(seed);
    const rootKey = await genRootPrivateKey(entropy);
    
    // when we are testing we want to use the same wallet every time, so we only need to fund one address, we will use index 0 for our keys and addresses to make things simple.
    const accountAddressKeyPrv = await genAddressPrv(rootKey, 0, 0, 0);

    // now we have our private key, we can generate a `Testnet` address
    const addressTest = Address.fromXPrv(rootKey, 'testnet')
    console.log(addressTest.toString())

    console.log(addressTest.paymentCreds.hash.toString())

    // aside from logging them, we will also return an object with our address, pubKeyHash and signingKey so we can use them in our transactions
    return {
        address: addressTest.toString(),
        pkh: addressTest.paymentCreds.hash.toString(),
        XPrv: accountAddressKeyPrv,
    }
}
```

Now we can send some testnet ADA to this wallet to fund it, so we can start writing and sending transactions, and we have a way to get those wallet details as needed during the transaction building process.

Return to [Getting Started](/docs/getting-started) to continue ...