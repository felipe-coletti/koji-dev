# Changelog

## Unreleased

-   Lists have `map`, `filter`, `reduce`, `any`, `all` and `find`, which take a function whose
    parameters need no types: `numbers.filter((x) => x > 0).map((x) => x * 2)`.
-   `regex` has look-around: `(?=...)`, `(?!...)`, `(?<=...)` and `(?<!...)`, a look-behind of any
    length, with matching still proportional to the length of the text.
-   The license is the Apache License 2.0 with LLVM Exceptions: a program compiled by Koji, which
    has its runtime and standard library built in, may be distributed without the license or the
    notice of Koji. NOTICE names the data of others in the standard library (the certificate
    authorities of Mozilla, the Unicode Character Database).
-   The parameters of an anonymous function given to a generic function or method take the types
    its other arguments tell: `result.mapErr((e) => ...)` and `apply((x) => x + 1, 4)` need no
    types written.
-   On macOS, Koji's linker links the libraries of `foreign` blocks too: a `.dylib` (from
    `LIBRARY_PATH`, Homebrew or `/usr/local/lib`), or the stub in the SDK of a library of the
    system. Building a program with `foreign` blocks no longer needs the linker of the system.

## 0.1.0 (preview)

The first release of Koji, a statically typed language that aims to be simple, readable and
fast, compiled to machine code by a compiler written in Koji. It is a preview: try it, and tell us
what is missing or wrong. Expect breaking changes before 1.0.

```koji
import http
import task

type Shape = {
    Circle(radius: decimal)
    Square(side: decimal)
}

function area(shape: Shape): decimal {
    return match (shape) {
        Circle(radius) => 3.14 * radius * radius
        Square(side) => side * side
    }
}

// Errors are values: `?` passes them on.
function half(text: string): Result[int, ParseError] {
    immut n = int(text)?
    return Ok(n / 2)
}

print(area(Circle(2)), area(Square(1.5)), 0.1 + 0.2) // 12.56 2.25 0.3
print(half('42'), half('x')) // Ok(21) Err(InvalidNumber('x'))

// Requests at the same time, on one thread.
immut pages = task.all([
    () => http.get('https://example.com'),
    () => http.get('https://example.org'),
])
```

### Getting it

Download the package of your platform below, unpack it and put its directory on your `PATH`:

| Platform | Package |
| --- | --- |
| Linux, x86-64 | `koji-0.1.0-linux-x64.tar.gz` |
| Linux, arm64 | `koji-0.1.0-linux-arm64.tar.gz` |
| macOS, Apple silicon | `koji-0.1.0-macos-arm64.tar.gz` |
| macOS, Intel | `koji-0.1.0-macos-x64.tar.gz` |
| Windows, x86-64 | `koji-0.1.0-windows-x64.zip` |

```sh
tar -xzf koji-0.1.0-linux-x64.tar.gz
export PATH=$PWD/koji-0.1.0-linux-x64:$PATH
kojic run hello.kj
```

`SHA256SUMS` has the checksums of the packages. On macOS, a downloaded `kojic` is quarantined:
`xattr -d com.apple.quarantine koji-0.1.0-macos-arm64/kojic` lets it run. Koji also builds from
its repository with nothing but `gzip` (`bootstrap/build.sh`, or `bootstrap\build.cmd` on Windows):
see [Getting Started](https://github.com/koji-language/koji/blob/main/doc/getting-started.md).

### The language

-   Strong, static typing with inference: `int`, `float`, `decimal` (exact, the default of
    literals with a point), `string`, `char`, `byte`, `boolean`, lists, maps, sets, records and
    sum types.
-   `match` with guards and exhaustiveness, `if` as an expression, no `null` (`Option`), errors as
    values (`Result` and `?`), panics for bugs, no exceptions.
-   Value semantics with copy-on-write, move-only types with `drop`, `mut` and `take` parameters.
-   Generics with the `equatable` and `orderable` constraints, methods, closures that capture a copy.
-   Modules with mandatory `export`, opaque types, packages from repositories of git (`koji.mod`,
    `koji.sum`, minimal version selection).
-   Tasks in groups, channels and shared values, on one thread; no call blocks the other tasks,
    and a task that computes is preempted.
-   `test` blocks and `kojic test`; `foreign` blocks for the functions of libraries of the system.

### The standard library

`cli`, `datetime`, `file`, `hash`, `http` (a client with TLS 1.3 written in Koji, and a server),
`json`, `math`, `path`, `regex`, `system` (programs run with an input, a directory and an
environment), `task` and `time`.

### The compiler

-   `kojic` is written in Koji and builds itself from a seed of its own executables: building
    Koji needs no C compiler, no Rust, nothing but `gzip`. Its own tests are written in Koji too.
-   It generates machine code itself (x86-64 and arm64) and links executables with its own linker
    (ELF, Mach-O and PE), the shared libraries of `foreign` blocks included on Linux and Windows.
-   `kojic run`, `build`, `check`, `test`, `lsp` (a language server), `init`, `add`, `update`,
    `version`. A [VS Code extension](https://github.com/koji-language/koji-vscode) gives the
    highlighting and the language server; the TextMate grammar is in `editors/textmate/`.

### Platforms

| Platform | Tested |
| --- | --- |
| Linux, x86-64 | the whole test suite, in CI |
| Linux, arm64 | the whole test suite, in CI |
| macOS, arm64 and x86-64 | the whole test suite, in CI (macOS 15) |
| Windows, x86-64 | the programs of the test suite, in CI (Windows Server 2022), and under Wine |

### Known limits

-   Tasks run on one thread.
-   `regex` has no look-around.
-   The parameters of an anonymous function given to a generic method (`.mapErr((e) => ...)`)
    need their type written.
-   On macOS, the libraries of `foreign` blocks are linked by the linker of the system.
-   The code is about 1.3 times as slow as C compiled by gcc (measured on the compiler itself).

### Feedback

Issues and ideas are welcome at https://github.com/koji-language/koji/issues.
