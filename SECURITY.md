# Security

## Supported versions

Koji is a preview: only its last release, and the `main` branch, get fixes. A fix comes in a new
release; versions before 1.0 are not patched.

## Reporting a vulnerability

Report it privately, through GitHub: on the
[Security tab](https://github.com/koji-language/koji/security) of the repository, choose
**Report a vulnerability**. Please do not open a public issue, pull request or discussion for it.

Say what is affected (the compiler, the runtime, a module of the standard library, the package
manager), the version (`kojic version`) and the platform, and, if you can, a small program that
shows it. You can expect an answer within a week, and to hear how the fix goes; once it is
released, the advisory credits you, unless you would rather it did not.

## What counts

Among others:

-   a program of Koji that, without a `foreign` block, reads or writes memory it should not, or runs
    code it should not;
-   `http` (TLS included) accepting a certificate it should refuse, or a connection an attacker can
    read or change;
-   `json`, `regex`, `http` or another module that parses data crashing the program in a way that is
    not a panic, or taking time or memory out of proportion to its input;
-   `kojic` itself: the package manager fetching or running code a `koji.sum` or a `koji.mod` does not
    allow, or a package with `foreign` blocks used without `allow foreign`;
-   the packages of a release that are not what its tag builds.

A panic on a bug of the program (an index out of range, an overflow) is the language working as
it should, not a vulnerability. What the functions of C of a `foreign` block do is outside what
Koji checks.
