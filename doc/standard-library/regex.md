# Regex

The `regex` module provides regular expressions for searching, validation, and string manipulation. `Regex` is an [opaque type](../modules-and-imports.md#opaque-types): it is made by `compile` only, and shown as `<regex '[0-9]+'>`.

## Import

```koji
import regex
```

## Compiling a Pattern

```koji
regex.compile(pattern: string) -> Result[Regex, RegexError]
```

A pattern is compiled once, and the resulting `Regex` is used as many times as needed. Only compiling can fail; the methods of `Regex` cannot.

```koji
type RegexError = {
    InvalidPattern(pattern: string, message: string) // the syntax is wrong: '(abc'
    Unsupported(pattern: string, message: string) // valid, but not supported: backreferences, a capture group in a look-around
    TooLarge(pattern: string) // too large to compile: 'a{1000}{1000}'
}
```

```koji
match (regex.compile('(abc')) {
    Ok(_) => print('valid')
    Err(error) => print(error) // invalid regular expression '(abc': unclosed group
}

// In a function that returns a Result, `?` is shorter:
function countWords(text: string): Result[int, regex.RegexError] {
    immut word = regex.compile('[a-z]+')?

    return Ok(word.findAll(text).length())
}
```

## Methods of `Regex`

| Method | Signature | Description |
| --- | --- | --- |
| `test` | `test(input: string) -> boolean` | Whether the pattern matches anywhere in `input` (use `^` and `$` to match the whole string) |
| `find` | `find(input: string) -> Option[Match]` | The first match |
| `findAll` | `findAll(input: string) -> Match[]` | Every match |
| `replace` | `replace(input: string, repl: string) -> string` | Replaces the first match with `repl` |
| `replaceAll` | `replaceAll(input: string, repl: string) -> string` | Replaces every match with `repl` |
| `split` | `split(input: string) -> string[]` | Splits `input` at every match |

In `repl`, `$1` (or `${1}`) is the text of the first group, `${name}` the text of a named group `(?P<name>...)`, and `$$` a literal `$`.

**Example:**

```koji
import regex

immut digits = regex.compile('[0-9]+')?

print(digits.test('abc123')) // true
print(digits.test('abc')) // false

match (digits.find('Order number: 12345')) {
    Some(found) => print(found.value) // 12345
    None => print('no number')
}

for found in digits.findAll('a1b22c333') {
    print(found.value)
}
// 1
// 22
// 333

print(digits.replace('a1b2', '#')) // a#b2
print(digits.replaceAll('a1b2', '#')) // a#b#
print(digits.split('a1b22c')) // ['a', 'b', 'c']

immut date = regex.compile('([0-9]{4})-([0-9]{2})')?

print(date.replaceAll('2025-10', '$2/$1')) // 10/2025
```

## Type: Match

Represents a found match.

```koji
Match {
	value: string
	start: int
	end: int
}
```

-   `start` and `end` are character positions (`end` is exclusive).

## Notes

-   The regex syntax is Perl-style (character classes, groups, quantifiers, anchors, look-around). Backreferences are not supported.
-   Look-around checks what comes before or after a position without making it part of the match: `(?=...)` (followed by), `(?!...)` (not followed by), `(?<=...)` (preceded by) and `(?<!...)` (not preceded by). `\d+(?= dollars)` finds the numbers followed by ` dollars`, and `(?<=\$)\d+` those after a `$`. A look-behind can have any length (`(?<=ab+)c`, `(?<=^|,)\w+`), unlike in many other languages. A group in a look-around does not capture: write `(?:...)` there (a capture group gives `Unsupported`).
-   The module is written in Koji (`std/regex.kj`). A pattern is compiled for a Pike machine, so matching takes a time proportional to the length of the text (times the size of the pattern), never exponential: there is no backtracking. Look-around keeps it so: each look-around is answered for every position of the text in one pass over it. The first match is the leftmost one, and among those the one the pattern prefers (the first branch of `|`, the longest repetition unless it is lazy). After a match, an empty match right at its end is skipped.
-   Supported: classes with ranges, `[^...]`, nested classes, `&&`, `--` and `~~`, `[[:alpha:]]` (ASCII); `\d`, `\w`, `\s` and their complements, which follow Unicode; `\p{L}` (`Letter`, `Alphabetic`), `\p{Lu}` (`Uppercase`), `\p{Ll}` (`Lowercase`), `\p{N}` (`Number`), `\p{Nd}` (`Digit`), `\p{White_Space}` and their `\P` complements; `^`, `$`, `\A`, `\z`, `\b`, `\B`, `\<`, `\>`; `(?=...)`, `(?!...)`, `(?<=...)`, `(?<!...)`; `(?:...)`, `(?P<name>...)`, `(?<name>...)`; the flags `i`, `m`, `s`, `x` and `U`, as `(?i)` or `(?i:...)`; `*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}` and their lazy forms (`*?`); the escapes `\n`, `\t`, `\x41`, `\x{1F600}`, `\u0041` and any escaped punctuation.
-   Other Unicode properties (`\p{Greek}`) are not supported: compiling gives `Unsupported`. Outside ASCII, `\d` and `\p{Nd}` match every numeric character, and `\w` letters and numbers (and `_`), as `isNumeric` and `isAlphaNumeric` do.
-   A pattern is too large (`TooLarge`) when it would compile to more than 500,000 instructions: `a{1000}{1000}`.
-   To learn more about regex syntax, visit: [https://regex101.com/](https://regex101.com/)
