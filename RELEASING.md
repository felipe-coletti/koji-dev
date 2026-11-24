# Releasing Koji

1.  The version is `version` in `compiler/main.kj` (`kojic version` prints it): set it, and write
    its section in `CHANGELOG.md` (`## <version>`, whose text becomes the notes of the release).
2.  Check that everything passes: `sh bootstrap/build.sh`, then `build/kojic test test/*.kj`, and
    CI, which runs the tests on every platform (Linux on x86-64 and arm64, macOS on arm64 and x86-64, Windows) and must be green.
3.  `bootstrap/package.sh` makes the packages in `dist/` locally, to try them before (CI makes them
    too, on each push, as the artifact `koji-packages`).
4.  Push the tag `v<version>` (`git tag v0.1.0 && git push origin v0.1.0`): the workflow
    `.github/workflows/release.yml` builds, tests and packs Koji again, and makes a draft release
    with the packages, `SHA256SUMS` and the notes. Read it, then publish it on GitHub. Do not
    create the release on GitHub by hand before: the workflow would make a second one, a draft
    with the packages, and the one created by hand would have none.
5.  The site ([koji-website](https://github.com/koji-language/koji-website)) is built every day
    with the last release and the documentation of its tag: the new version is there by the next
    day, or at once by running its workflow Pages by hand.

CI's jobs that GitHub cancels for want of a runner (it happens to macOS on arm64), before any step,
are run again by `.github/workflows/rerun.yml`, up to the third attempt. GitHub runs a workflow of
`workflow_run` only from the default branch: it works once it is on `main`.
