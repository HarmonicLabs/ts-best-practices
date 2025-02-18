# styling

## Indentation

**ALWAYS** 4 spaces.

> _**NOTE:**_ In VSCode if you are in a file that uses 2 spaces,
> to quickly fix _most_ of the indenting you can
> set "indent using spaces" to 4
> and then press `Ctrl+Shift+I`

DO NOT use tabs, since different editors render them differently (for example github renders tabs as 8 spaces).

you can of course use tabs while writing code, just remember to set "indent using spaces" in your editor, and set it to 4 spaces.

> only excepiton acceptable in front-end repos, where 2 spaces are ok

[here is an example](./wrong_indentation_example.md) highlighting the reasoning behind this decision.

## Vertical code (avoid nesting)

Main reason behind the 4 spaces decision is that it makes easier to spot convoluted code.

every level of indentation is a level of complexity that may or may not execute.

It makes code hard to follow.

As a rule of thumb, at 2 levels of indentation you should start questioning if you really need that second layer; at 3 levels, almost surely you don't need the 3rd layer.

### use early returns

in general if you have something like

(see the [documentation for `>>>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Unsigned_right_shift) if you don't know what it does)

(this functions is bad performance-wise, only educational purposes)
```ts
function u32( thing: number ): number
{
    if( thing < 0 )
    {
        return u32( -thing );
    }
    else
    {
        return thing >>> 0;
    }
}
```

you can omit the `else`, because the if branch early returns, saving 1 indentation.

```ts
function u32( thing: number ): number
{
    if( thing < 0 )
    {
        return u32( -thing );
    }

    return thing >>> 0;
}
```

### swap if then else branches

#### [example](../examples/asmBadParser.md)

especially used together with early returns, if you see that the else branch has less indentation
than the if branch, negate the condition, swap the branches, early return:

```ts
if( condition )
{
    while( a ) {
        while( b ) {
            /** ... do stuff */
        }
    }
}
else
{
    /** do other things */
}
return;
```

becomes
```ts
if( !condition )
{
    /** do other things */
    return;
} 

while( a ) {
    while( b ) {
        /** ... do stuff */
    }
}
return;
```

## `try` `catch` **MUST** handle (and recover) the error; or else don't use it.

```ts
let stuff: any
try {
    stuff = maybeFails();
} catch (e) {
    throw new Error("I'm failing here: " + e.message);
}
```

Instead just let the error go up if it really needs to be thrown.

```ts
const stuff = maybeFails();
```

and with that we just turned 6 lines into a single one.