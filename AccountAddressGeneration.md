# buildooor Library Documentation: Generating Cardano Accounts and Addresses

## Introduction

This documentation explains how to use the **buildooor** library in conjunction with **bip39** to generate mnemonic phrases, private keys, and addresses for the Cardano blockchain. Generating mnemonic phrases and private keys on Cardano follows a similar process to other blockchains that utilize hierarchical account/key generation. Using a single seed phrase, you can generate multiple accounts, and for each account, you can generate multiple addresses. In Cardano's case, there are also different types of addresses, but this documentation focuses on **base addresses** and **stake addresses**.

Most modern wallets simplify the process by generating only one address per account. However, since an account can have multiple addresses, each with its own private key, managing transactions can become cumbersome. For example, if you have two addresses under the same account (e.g., `address0` with 20 ADA and `address1` with 20 ADA), sending 30 ADA to a friend would require including both addresses' UTXOs as inputs and providing the signing keys for both addresses during the transaction. This complexity is why many wallets and implementations opt for a single-address-per-account approach.

Before generating any accounts or addresses, you need a **mnemonic phrase** (sometimes called a seed phrase), which is generated using the **BIP39** standard. If you already have a seed phrase, you can verify and use it as well.

## Prerequisites

Before you begin, ensure you have the necessary libraries installed:

```bash
npm i @harmoniclabs/buildooor
npm i bip39
```

## Imports

Import the required functions and types:

```typescript
import { generateMnemonic, mnemonicToEntropy, validateMnemonic } from 'bip39';
import { XPrv, harden, Address, StakeAddress, NetworkT } from '@harmoniclabs/buildooor';
```

## Generating Mnemonic Phrases

Mnemonic phrases (or seed phrases) are used to generate the root private key from which all other keys are derived.

### `genSeedPhrase`

```typescript
/**
 * Generates a new mnemonic phrase with 256 bits of entropy.
 * @returns {string | object} The generated mnemonic phrase, or an error object if generation fails.
 */
export function genSeedPhrase(): string | object {
  try {
    const mnemonic = generateMnemonic(256);
    return mnemonic;
  } catch (error) {
    console.error(error);
    return error;
  }
}
```

### `validateSeedPhrase`

```typescript
/**
 * Validates a given mnemonic phrase.
 * @param {string} seed - The mnemonic phrase to validate.
 * @returns {boolean | object} True if valid, false if invalid, or an error object if an exception occurs.
 */
export function validateSeedPhrase(seed: string): boolean | object {
  try {
    const isValid = validateMnemonic(seed);
    return isValid;
  } catch (error) {
    console.log(error);
    return error;
  }
}
```

### `seedPhraseToEntropy`

```typescript
/**
 * Converts a mnemonic phrase to its corresponding entropy.
 * @param {string} seed_phrase - The mnemonic phrase.
 * @returns {string} The entropy as a hexadecimal string.
 * @throws {Error} If the mnemonic is invalid.
 */
export function seedPhraseToEntropy(seed_phrase: string): string {
  return mnemonicToEntropy(seed_phrase);
}
```

## Generating Private Keys

From the mnemonic phrase, you can generate the root private key and then derive account and address private keys.

### `genRootPrivateKey`

```typescript
/**
 * Generates the root private key from the given entropy.
 * @param {string} entropy - The entropy as a hexadecimal string.
 * @returns {XPrv | string} The root private key if successful, or 'root key error' if an error occurs.
 */
export function genRootPrivateKey(entropy: string): XPrv | string {
  try {
    const xprv_root = XPrv.fromEntropy(entropy, '');
    return xprv_root;
  } catch (error) {
    console.log('root key error: ', error);
    return 'root key error';
  }
}
```

### `genAccountPrivatekey`

```typescript
/**
 * Generates the account private key from the root private key and account index.
 * @param {XPrv} rootKey - The root private key.
 * @param {number} index - The account index.
 * @returns {XPrv} The derived account private key.
 */
export function genAccountPrivatekey(rootKey: XPrv, index: number): XPrv {
  const accountKey = rootKey
    .derive(harden(1852)) // purpose
    .derive(harden(1815)) // coin type
    .derive(harden(index)); // account index
  return accountKey;
}
```

### `genAddressPrv`

```typescript
/**
 * Generates the address private key from the root private key, account index, address type, and address index.
 * @param {XPrv} xprv_root - The root private key.
 * @param {number} accIndex - The account index.
 * @param {number} addressType - The address type (e.g., 0 for external, 1 for change).
 * @param {number} addressIndex - The address index.
 * @returns {XPrv} The derived address private key.
 */
export function genAddressPrv(xprv_root: XPrv, accIndex: number, addressType: number, addressIndex: number): XPrv {
  return xprv_root
    .derive(harden(1852))
    .derive(harden(1815))
    .derive(harden(accIndex))
    .derive(addressType)
    .derive(addressIndex);
}
```

### `genAddressPrivateKey`

```typescript
/**
 * Generates the spending private key from the account private key and index.
 * This corresponds to the derivation path m/1852'/1815'/accIndex'/0/index
 * @param {XPrv} accountKey - The account private key.
 * @param {number} index - The address index.
 * @returns {XPrv} The derived spending private key.
 */
export function genAddressPrivateKey(accountKey: XPrv, index: number): XPrv {
  const spendingKey = accountKey
    .derive(0) // external
    .derive(index);
  return spendingKey;
}
```

### `genAddressStakeKey`

```typescript
/**
 * Generates the stake private key from the account private key and index.
 * This corresponds to the derivation path m/1852'/1815'/accIndex'/2/index
 * @param {XPrv} accountKey - The account private key.
 * @param {number} index - The stake index.
 * @returns {XPrv} The derived stake private key.
 */
export function genAddressStakeKey(accountKey: XPrv, index: number): XPrv {
  const stakeKey = accountKey
    .derive(2) // stake key
    .derive(index);
  return stakeKey;
}
```

## Generating Addresses

You can generate base and stake addresses directly from entropy or from derived private keys.

### `genBaseAddressFromEntropy`

```typescript
/**
 * Generates a base address from entropy, network, account index, and address index.
 * @param {string} entropy - The entropy as a hexadecimal string.
 * @param {NetworkT} network - The network type (e.g., 'mainnet', 'testnet').
 * @param {number} accountIndex - The account index.
 * @param {number} addressIndex - The address index.
 * @returns {Address} The generated base address.
 */
export function genBaseAddressFromEntropy(entropy: string, network: NetworkT, accountIndex: number, addressIndex: number): Address {
  const addressFromEntropy = Address.fromEntropy(entropy, network, accountIndex, addressIndex);
  const baseAddress = new Address({
    network,
    paymentCreds: addressFromEntropy.paymentCreds,
    stakeCreds: addressFromEntropy.stakeCreds,
    type: 'base',
  });
  return baseAddress;
}
```

### `genStakeAddressFromEntropy`

```typescript
/**
 * Generates a stake address from entropy, network, account index, and address index.
 * @param {string} entropy - The entropy as a hexadecimal string.
 * @param {NetworkT} network - The network type (e.g., 'mainnet', 'testnet').
 * @param {number} accountIndex - The account index.
 * @param {number} addressIndex - The stake index.
 * @returns {StakeAddress} The generated stake address.
 */
export function genStakeAddressFromEntropy(entropy: string, network: NetworkT, accountIndex: number, addressIndex: number): StakeAddress {
  const addressFromEntropy = Address.fromEntropy(entropy, network, accountIndex, addressIndex);
  const stakeAddress = new StakeAddress({
    network,
    credentials: addressFromEntropy.stakeCreds.hash,
    type: 'stakeKey',
  });
  return stakeAddress;
}
```
