# Releasing Koji

1.  The version is `version` in `compiler/main.kj` (`kojic version` prints it): set it, and write
    its section in `CHANGELOG.md` (`## <version>`, whose text becomes the notes of the release).
2.  Check that everything passes: `sh bootstrap/build.sh`, then `build/kojic test test/*.kj`, and
    CI, which runs the tests on every platform (Linux on x86-64 and arm64, macOS on arm64 and x86-64, Windows) and must be green.
3.  `bootstrap/package.sh` makes the packages in `dist/` locally, to try them before (CI makes them
    too, on each push, as the artifact `koji-packages`).
4.  Make the tag `v<version>`, one of two ways; either way, the workflow
    `.github/workflows/release.yml` builds, tests and packs Koji again (some 10 minutes):
    -   with a release on GitHub (Releases, Draft a new release, `v0.2.1` as a new tag on `main`,
        Publish release): the packages and `SHA256SUMS` are added to it, and its notes are those
        of `CHANGELOG.md` if it was published with none;
    -   or pushed (`git tag v0.2.1 && git push origin v0.2.1`): the workflow makes a draft release
        with the packages and the notes, to read and publish on GitHub.
5.  The site ([koji-website](https://github.com/koji-language/koji-website)) is built every day
    with the last release and the documentation of its tag: the new version is there by the next
    day, or at once by running its workflow Pages by hand.

CI's jobs that GitHub cancels for want of a runner (it happens to macOS on arm64), before any step,
are run again by `.github/workflows/rerun.yml`, up to the third attempt. GitHub runs a workflow of
`workflow_run` only from the default branch: it works once it is on `main`.
