# optimizations

## Avoid closures, prefer binding

where do you find closures? sometimes you could find them in classes implementations where you would want to have a "truly" private field.

example
```ts
class MyThing
{
    public thing!: number

    constructor()
    {
        let _trulyPrivateThing = 0;
        
        Object.defineProperty(
            this, "thing", {
                // this is a closure
                get: () => _trulyPrivateThing,
                // this is a closure
                set: ( next: any ) => {
                    if( Number.isSafeInteger( next ) )
                        _trulyPrivateThing = next;
                },
                enumerable: true,
                configurable: false
            }
        );
    }
}
```

The problem with closures is that they need to remember the scope outside their definition.

They might be handy, but, especially in large contexts (large here being even 2-3 variables holidng objects), memory really doesn't like them.

The **absolute best thing to do** here, even though not strictly semantically the same, would be to use getters and setters.

```ts
class MyThing
{
    private _notReallyPrivateThing!: number
    get thing(): number
    {
        return this._notReallyPrivateThing;
    }
    set thing( next: any )
    {
        if( Number.isSafeInteger( next ) )
            this._notReallyPrivateThing = next;
    }

    constructor()
    {
        this._notReallyPrivateThing = 0;
    }
}
```

The above is the best solution in terms of performance, because it only defines a single getter/setter, for **every** instance of `MyThing`.

However this relies on `_notReallyPirvateThing` being, in reality, accessible at js runtime.

This should **RARELY** be an issue.

If you are **ABSOLUTELY SURE** that you want a clousure, the best way to implement it is via `bind` and explicit `this`

```ts
class MyThing
{
    public thing!: number

    constructor()
    {
        const thingScope: ThingScope = { _thing: 0 };

        Object.defineProperty(
            this, "thing", {
                // this is a closure
                get: getThing.bind( thingScope ),
                // this is a closure
                set: setThing.bind( thingScope ),
                enumerable: true,
                configurable: false
            }
        );
    }
}

interface ThingScope {
    _thing: number
}

function getThing( this: ThingScope ): number
{
    return this._thing;
}

function setThing( this: ThingScope, next: number )
{
    if( Number.isSafeInteger( next ) )
        this._thing = next;
}
```

in this way, `_thing` is truly private, the binded function DOES NOT save the entire scope, **BUT** you still allocate a new function, so it takes a bit more memory than the optimal example of before. Still better than the entire scope though.