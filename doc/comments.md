# Comments

## Single-line Comments

Everything after `//` until the end of the line is a comment.

```koji
// Single-line comment
print('ok') // a comment after code
```

`//` is always a comment: integer division uses `/` (see [Operators](operators.md)).

## Multi-line Comments

```koji
/*
Multi-line
comment
*/
```

## Documentation

A comment on its own lines right above a declaration (a function, a type, a variable), with no blank line between them, documents it: the language server shows it with the type of the name, in the editor, wherever the name is used, in the file, in a module that imports it, and for the functions of the standard library.

```koji
// The price with the tax of `rate` (0.1 for 10%).
function withTax(price: decimal, rate: decimal): decimal {
    return price * (1 + rate)
}
```

The comment is shown as Markdown: `code` in backticks, *emphasis*, lists. Nothing else marks it: there is no special syntax for documentation, only the place of the comment.
