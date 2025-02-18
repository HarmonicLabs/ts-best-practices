
## this code comes from the [Assemblyscript parser](https://github.com/AssemblyScript/assemblyscript/blob/513acc8b7db01621aedfc78c295de62a9e7ca560/src/parser.ts#L370-L410)

this is what we are working with

```ts
// handle plain exports
if (flags & CommonFlags.Export) {
  if (defaultEnd && tn.skipIdentifier(IdentifierHandling.Prefer)) {
    if (declareEnd) {
      this.error(
        DiagnosticCode.An_export_assignment_cannot_have_modifiers,
        tn.range(declareStart, declareEnd)
      );
    }
    statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);
    defaultStart = defaultEnd = 0; // consume
  } else {
    statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
  }

// handle non-declaration statements
} else {
  if (exportEnd) {
    this.error(
      DiagnosticCode._0_modifier_cannot_be_used_here,
      tn.range(exportStart, exportEnd), "export"
    ); // recoverable
  }
  if (declareEnd) {
    this.error(
      DiagnosticCode._0_modifier_cannot_be_used_here,
      tn.range(declareStart, declareEnd), "declare"
    ); // recoverable
  }
  if (namespace) {
    this.error(
      DiagnosticCode.Namespace_can_only_have_declarations,
      tn.range(startPos)
    );
  } else {
    statement = this.parseStatement(tn, true);
  }
}
break;
```

first step is to [indent using 4 spaces](../styling/README.md#indentation).

after that we inverse (boolean not) the first if condition and we swap the two bodies 

```ts
// handle plain exports
if(!( flags & CommonFlags.Export ))
{
    if (exportEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(exportStart, exportEnd), "export"
        ); // recoverable
    }
    if (declareEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(declareStart, declareEnd), "declare"
        ); // recoverable
    }
    if (namespace) {
        this.error(
            DiagnosticCode.Namespace_can_only_have_declarations,
            tn.range(startPos)
        );
    } else {
        statement = this.parseStatement(tn, true);
    }
}
else
{
    if (defaultEnd && tn.skipIdentifier(IdentifierHandling.Prefer)) {
        if (declareEnd) {
            this.error(
                DiagnosticCode.An_export_assignment_cannot_have_modifiers,
                tn.range(declareStart, declareEnd)
            );
        }
        statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);
        defaultStart = defaultEnd = 0; // consume
    } else {
        statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
    }

// handle non-declaration statements
}
break;
```

now we can [early return the first body](../styling/README.md#use-early-returns) and remove the current else branch.

```ts
// handle plain exports
if(!( flags & CommonFlags.Export ))
{
    if (exportEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(exportStart, exportEnd), "export"
        ); // recoverable
    }
    if (declareEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(declareStart, declareEnd), "declare"
        ); // recoverable
    }
    if (namespace) {
        this.error(
            DiagnosticCode.Namespace_can_only_have_declarations,
            tn.range(startPos)
        );
    } else {
        statement = this.parseStatement(tn, true);
    }

    // early return
    break;
}

if (defaultEnd && tn.skipIdentifier(IdentifierHandling.Prefer)) {
    if (declareEnd) {
        this.error(
            DiagnosticCode.An_export_assignment_cannot_have_modifiers,
            tn.range(declareStart, declareEnd)
        );
    }
    statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);
    defaultStart = defaultEnd = 0; // consume
} else {
    statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
}
break;
```

in the body of the if branch we can early return each of the inner if statemets, and remove te last `else` 

```ts
// handle plain exports
if(!( flags & CommonFlags.Export ))
{
    if (exportEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(exportStart, exportEnd), "export"
        );
        break; // early return
    }
    if (declareEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(declareStart, declareEnd), "declare"
        ); 
        break; // early return
    }
    if (namespace) {
        this.error(
            DiagnosticCode.Namespace_can_only_have_declarations,
            tn.range(startPos)
        );
        break; // early return
    }

    // inlined else body
    statement = this.parseStatement(tn, true);

    break; // early return
}
```

and now we work with the remaining code outside the first if statement

currently:
```ts
if (defaultEnd && tn.skipIdentifier(IdentifierHandling.Prefer)) {
    if (declareEnd) {
        this.error(
            DiagnosticCode.An_export_assignment_cannot_have_modifiers,
            tn.range(declareStart, declareEnd)
        );
    }
    statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);
    defaultStart = defaultEnd = 0; // consume
} else {
    statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
}
break;
```

We see that the if branch has an inner indentation, while the else branch has none.

for this reaso we invert the condition (boolean not) and we early return, removing the else branch.

while we are at it, we break also the condition on more lines

```ts
if(!(
    defaultEnd
    && tn.skipIdentifier(IdentifierHandling.Prefer)
))
{
    statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
    break; // early return
}

if (declareEnd) {
    this.error(
        DiagnosticCode.An_export_assignment_cannot_have_modifiers,
        tn.range(declareStart, declareEnd)
    );
}

statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);

defaultStart = defaultEnd = 0;

break;
```

the new code now looks like this:

```ts
// handle plain exports
if(!( flags & CommonFlags.Export ))
{
    if (exportEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(exportStart, exportEnd), "export"
        );
        break; // early return
    }
    if (declareEnd) {
        this.error(
            DiagnosticCode._0_modifier_cannot_be_used_here,
            tn.range(declareStart, declareEnd), "declare"
        ); 
        break; // early return
    }
    if (namespace) {
        this.error(
            DiagnosticCode.Namespace_can_only_have_declarations,
            tn.range(startPos)
        );
        break; // early return
    }

    // inlined else body
    statement = this.parseStatement(tn, true);

    break; // early return
}

if(!(
    defaultEnd
    && tn.skipIdentifier(IdentifierHandling.Prefer)
))
{
    statement = this.parseExport(tn, startPos, (flags & CommonFlags.Declare) != 0);
    break; // early return
}

if (declareEnd) {
    this.error(
        DiagnosticCode.An_export_assignment_cannot_have_modifiers,
        tn.range(declareStart, declareEnd)
    );
}

statement = this.parseExportDefaultAlias(tn, startPos, defaultStart, defaultEnd);

defaultStart = defaultEnd = 0;

break;
```


