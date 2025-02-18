# `normalizeITxBuildArgsInputs`

this example comes from fixing [this commit in `buildoor`](https://github.com/HarmonicLabs/buildooor/blob/74d8e7052fa36d2e554d8f5bf6d4eb432d74033f/src/txBuild/ITxBuildArgs.ts#L155)

here's what we start with:

```ts
//Check input type and convert to NormalizedITxBuildInput
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO  ): NormalizedITxBuildInput {
  if (typeof input === "string") {
    try {
      const cborData = fromHex(input);
      const iUtxo = UTxO.fromCbor(cborData);
      if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
      return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    } catch (error) {
      throw new Error("Error processing CBOR data: " + (error as Error).message);
    }
  } else if (input instanceof Uint8Array) {
    try {
      const iUtxo = UTxO.fromCbor(input);
      if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
      return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    } catch (error) {
      throw new Error("Error processing CBOR data: " + (error as Error).message);
    }
  } else if (isIUTxO(input)) {
    return { utxo: new UTxO(input) };
  } else {
    return normalizeITxBuildInput(input);
  }
}
```

First style improvements: [early returns]()

when an if alwasy terminates (either returns or throw) avoid elses, you get one less indentation


```ts
//Check input type and convert to NormalizedITxBuildInput
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO  ): NormalizedITxBuildInput {
  if (typeof input === "string") {
    try {
      const cborData = fromHex(input);
      const iUtxo = UTxO.fromCbor(cborData);
      if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
      return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    } catch (error) {
      throw new Error("Error processing CBOR data: " + (error as Error).message);
    }
  }
  
  if (input instanceof Uint8Array) {
    try {
      const iUtxo = UTxO.fromCbor(input);
      if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
      return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    } catch (error) {
      throw new Error("Error processing CBOR data: " + (error as Error).message);
    }
  }

  if (isIUTxO(input)) {
    return { utxo: new UTxO(input) };
  }

  return normalizeITxBuildInput( input )
}
```

This could have been even more evident if we [indented using 4 spaces](), so we fix that too

```ts
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO  ): NormalizedITxBuildInput {
    if (typeof input === "string") {
        try {
            const cborData = fromHex(input);
            const iUtxo = UTxO.fromCbor(cborData);
            if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
            return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
        } catch (error) {
            throw new Error("Error processing CBOR data: " + (error as Error).message);
        }
    }
    
    if (input instanceof Uint8Array) {
        try {
            const iUtxo = UTxO.fromCbor(input);
            if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
            return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
        } catch (error) {
            throw new Error("Error processing CBOR data: " + (error as Error).message);
        }
    }

    if (isIUTxO(input)) {
        return { utxo: new UTxO(input) };
    }

    return normalizeITxBuildInput( input )
}
```

by doing this we can easly spot some unnecesary indentation, and here we see that maybe we are using [`try``catch` inappropriately]()

```ts
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO  ): NormalizedITxBuildInput {
    if (typeof input === "string") {
        const cborData = fromHex(input);
        const iUtxo = UTxO.fromCbor(cborData);
        if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
        return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    }
    
    if (input instanceof Uint8Array) {
        const iUtxo = UTxO.fromCbor(input);
        if (!isIUTxO(iUtxo)) throw new Error("Invalid UTxO structure");
        return normalizeITxBuildInput({ utxo: new UTxO(iUtxo) });
    }

    if (isIUTxO(input)) {
        return { utxo: new UTxO(input) };
    }

    return normalizeITxBuildInput( input )
}
```

Then we [use the types]() to see that `isIUTxO` and `new UTxO` calls are doing duplicate works, since `UTxO.fromCbor` already returns an instance of the `UTxO` class.

```ts
//Check input type and convert to NormalizedITxBuildInput
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO  ): NormalizedITxBuildInput {
    if (typeof input === "string") {
        const cborData = fromHex(input);
        const iUtxo = UTxO.fromCbor(cborData);
        return normalizeITxBuildInput({ utxo: iUtxo });
    }
    
    if (input instanceof Uint8Array) {
        const iUtxo = UTxO.fromCbor(input);
        return normalizeITxBuildInput({ utxo: iUtxo });
    }

    if (isIUTxO(input)) {
        return { utxo: new UTxO(input) };
    }

    return normalizeITxBuildInput( input );
}
```

by doing this, we notice that both the cases `typeof input === "string"` and `input instanceof Uint8Array` are calling the same `UTxO.fromCbor` static method.

This means that we are duplicating code taht `UTxO.fromCbor` is already handling.

if we look at the type we see that `UTxO.fromCbor` handles `CanBeCborString`; where `CanBeCborString` is nothing more than:

```ts
type CanBeCborString = string | Uint8Array | CborString;
```

to avoid this repetition we can check for the type at runtime using the function `canBeCborString`  and merge the two cases:

```ts
//Check input type and convert to NormalizedITxBuildInput
function normalizeITxBuildArgsInputs(input: ITxBuildInput | IUTxO | CanBeCborString ): NormalizedITxBuildInput {
    if( canBeCborString( input ) ) {
        const cborData = forceCborString(input);
        const iUtxo = UTxO.fromCbor(cborData);
        return normalizeITxBuildInput({ utxo: iUtxo });
    }

    if (isIUTxO(input)) {
        return { utxo: new UTxO(input) };
    }

    return normalizeITxBuildInput( input );
}
```

finally, we fix some minor [inconsistencies with spacing](), and we [inline variables used once]():

```ts
// Check input type and convert to NormalizedITxBuildInput
function normalizeITxBuildArgsInputs( input: ITxBuildInput | IUTxO | CanBeCborString ): NormalizedITxBuildInput
{
    if( canBeCborString( input ) )
    return normalizeITxBuildInput({
        utxo: UTxO.fromCbor(
            forceCborString( input )
        )
    });

    if( isIUTxO(input) ) return { utxo: new UTxO(input) };

    return normalizeITxBuildInput( input );
}
```