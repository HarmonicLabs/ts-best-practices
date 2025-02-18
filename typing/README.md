# Typing

There are a few basic rules on typings:


## Avoid inheritance like the plague, use interfaces instead.

instead of
```ts
abstract class Parallelogram
{
    short: number;
    long: number;
    constructor(
        short: number,
        long: number
    )
    {
        this.short = short;
        this.long = long;
    }

    area(): number { return this.short * this.long; }
}

// bad where is the `area` method ?
class Rectangle extends Parallelogram {}

// bad where is the `area` method ?
class Square extends Parallelogram {
    constructor( side: number )
    {
        super( side, side );
    }
}
```

just do

```ts
interface IArea {
    area(): number
}

type Parallelogram = Rectangle | Square;

function isParallelogram( thing: any ): thing is Parallelogram
{
    return (
        thing instanceof Rectangle
        || thing instanceof Square
    );
}

class Rectangle
    implements IArea
{
    short: number;
    long: number;
    constructor(
        short: number,
        long: number
    )
    {
        this.short = short;
        this.long = long;
    }

    area(): number { return this.short * this.long; }
}

class Square
    implements IArea
{
    side: number;
    constructor( side: number )
    {
        this.side = side;
    }

    area(): number { return this.side * this.side; }
}
```

It is a little bit more code, but you know exactly where everyting is.

You'll thank me when you'll have to debug it.

## Typing functions

Take the **least descriptive** types as function inputs (but **NOT** `any`)

and return the **most descriptive** types as outputs

for example, if you are implementing a fixed output hash function
you may take as input anything you can convert to bytes;

but return stricly `Uint8Array` with length 32.

```ts
function sha2_256( data: string | number[] | Uint8Array ): Uint8Array & { length: 32 }
{
    if( Array.isArray( data ) ) data = Uint8Array.from( data );
    if(!(data instanceof Uint8Array)) data = fromHex( data.toString() );

    /* ... sha2_256 implementation ... */
}
```

## **ALWAYS** make explicit the return type

**BAD**
```ts
function getManyThings()
{
    const a = getA();
    const b = getB();
    const c = getC();
    const d = getD();

    return { a, b, c, d };
}
```

**GOOD**
```ts
interface ManyThings {
    a: A;
    b: B;
    c: C;
    d: D;
}

function getManyThings(): ManyThings
{
    const a = getA();
    const b = getB();
    const c = getC();
    const d = getD();

    return { a, b, c, d };
}
```

### why?

say you later use `getManyThings` in many places and do stuff like

```ts
function useManyThings(): void
{
    const things = getManyThings();

    console.log( things.a.getStrangeValue() );
    console.log( things.b.getStrangeValue() );
    console.log( things.c.getStrangeValue() );
    console.log( things.d.getStrangeValue() );
}
```

but at some point in the future you decide you no longer need `c`.

if you just typed the thing, typescript would have told you that `thing.c` does not exsist.

Instead, if you don't type the return value you'll get just a runtime error.

```
Uncaught TypeError: Cannot read properties of undefined (reading 'getStrangeValue')
```

## functions with side effects SHOULD return `void`

## spend time writing good types