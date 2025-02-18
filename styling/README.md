# styling

## Indentation

**ALWAYS** 4 spaces.

DO NOT use tabs, since different editors render them differently (for example github renders tabs as 8 spaces).

you can of course use tabs while writing code, just remember to set "indent using spaces" in your editor, and set it to 4 spaces.

> only excepiton acceptable in front-end repos, where 2 spaces are ok

[here is an example](./wrong_indentation_example.md) highlighting the reasoning behind this decision.

## Vertical code (avoid nesting)

Main reason behind the 4 spaces decision is that it makes easier to spot convoluted code.

### Early returns are encouraged


## `try` `catch`

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