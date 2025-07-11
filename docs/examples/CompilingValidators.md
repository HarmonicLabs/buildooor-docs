---
sidebar_position: 1
slug: '/examples/compiling-validators'
---

# Compiling Validators

The complexity in compiling validators really comes down to the validator params themselves.

Without spending too much time on explaining how validators work on Cardano, we are going to take a look at compiling validators with several examples that will help demonstrate how to create the various `UPLCConst`'s we may need in order to parameterise our validators correctly.

Validators compile to a language called `UPLC` so when we are parameterising our validators, we need to convert our parameters into that `UPLC` format.

We will start by creating a file called `makeValidator.ts`, this module will hold the functions that allow us to compile validators and return the validator details in a usable format for our transaction code.

We need to import a few things from `Buildooor`:

```ts
import {
  Script,
  ScriptType,
  UPLCTerm,
  parseUPLC,
  fromHex,
  Application,
  compileUPLC,
  UPLCProgram,
  UPLCConst,
} from "@harmoniclabs/buildooor";

...
```

The most common things we will need from a validator is the compiled validator as a `Script`, and the hash of that script - which is used for `Credentials`, `Addresses` and `PolicyId` of that validator.

Define a function called `makeValidator()` which will create the `Script` from our validator and apply any params as needed, which must be included as `UPLCConst[]` - we will go into this in more detail later.

```ts
// Validators often come in the from of a hexString when compiled,
// We need to parse it into a UPLC program,
// then we can apply any parameters to it
// makeValidator will do this AND return the validator with the hash as an object
export async function makeValidator(
  compiled: string,
  params: UPLCConst[]
) {
  // The validator will need to have its Plutus Version specified,
  // We are using Plutus V3 here
  const validator = new Script(
    ScriptType.PlutusV3,
    applyParams(compiled, params)
  );

  // the hash of the validator can be used to construct the address
  // or used as the PolicyID for minting
  // or as the credential for staking
  const hash = validator.hash;

  return { validator, hash };
}
```

We also need to define this `applyParams()` function, which will take the `compiled` validator code as a string, and our list of `UPLCConst` params:

```ts
// applyParams will take the hex string of a compiled validator and an array of parameters,
// and apply the parameters to the validator, returning the compiled code as a buffer
export function applyParams(compiledCode: string, params: UPLCConst[]) {
  // first we parse the compiled code into a UPLC program
  const program = parseUPLC(fromHex(compiledCode));
  
  // then we apply the parameters to the program body using applyMany
  const applied = compileUPLC(
    new UPLCProgram(program.version, applyMany(program.body, params))
  );
  
  return applied.toBuffer().buffer;
}
```

Finally, `applyMany()` function must also be defined. This function will apply each of the parameters in the list, in order and return the updated `UPLCProgram`.

```ts
// applyMany will take a UPLCTerm (the function) and an array of UPLCConst (the arguments),
// and apply each argument to the function, returning the final UPLCTerm
export function applyMany(script: UPLCTerm, params: UPLCConst[]) {
  for (let i = 0; i < params.length; i++) {
    script = new Application(script, params[i]);
  }

  return script;
}
```

Now we have these functions defined, we only need to worry about converting any params to the appropriate `UPLCConst` and pass them to `makeValidator()`.

Return to [Getting Started](/docs/getting-started) to continue ...