# Build the prebuilt lym only when a targeted package's tests use it

A `lym gate test` tier with `prebuilt_lym` built `lym` from `./cmd/lym` before every run, whether or not any test of its targets runs the binary.
The study in `docs/research/speed/gate-fixed-cost-2026-11-27/report.md` measured that build at 2.4 to 2.8 s of a tier1 call on one small package.
That call's total wall time was about 6 s.

The tier now builds `lym` only when a test it runs may use the binary.

## What changed

- `impactset.TestsUseLymBinary(dir)` reports whether the tests of the package in `dir` may run the `lym` binary.
  They may when a `_test.go` file in `dir` imports `internal/testkit/lymbin`, or when `dir` is that kit's own directory.
  It reads only the test files' imports, with no `go list`, and ignores build constraints, so a file only another platform compiles still counts.
  A directory or test file it cannot read or parse counts as using the binary.
- `gatecli` decides per tier run, from the invocations' targets before narrowing, whether to prebuild:
  - an invocation with no target, a target holding `...` (the whole set's `./...` included), or a target that is no directory path, such as an import path, builds it;
  - a directory target, relative to its invocation's run directory or absolute, builds it when `TestsUseLymBinary` says so;
  - otherwise the tier logs that it builds none and exports no `LYM_PREBUILT_LYM`.
- The same decision covers a `--copies` run, which still builds at most once for all its copies.
- The `gatecli` package doc, the `impactset` package doc and `pattern/PATTERN-test-speed.md` state the new condition.

## Design decisions

- **Every route to the binary was checked before narrowing.**
  The only reader of `LYM_PREBUILT_LYM` that hands a test the binary is `internal/testkit/lymbin`.
  The Testkit Invariant, enforced by `internal/testkit/enforcement_test.go`, keeps every non-test file outside `internal/testkit/` from importing a kit, and keeps every other kit from importing `lymbin`.
  So a test reaches the binary only through a test file that imports `lymbin` directly, or through `lymbin`'s own in-package tests.
  At this commit, 26 test files in 11 package directories import it, and every one of those directories counts under the new rule.
  The other tests that name the variable (`verifytree`, `cardverify`, `shellguard` and the `gatecli` fixtures) set it themselves or check that it is stripped.
  They never rely on the gate's value.
  A test that starts a child `go test` of a package that imports `lymbin` passes the variable on only when it was set.
  When it was not, the child's `lymbin` builds the binary itself: slower, never missing.
- **The decision reads the targets before `{test_targets}` narrowing.**
  The narrowing runs a `go list` after the prebuild, and the unnarrowed targets can only make the check build more often, never less.
- **Uncertain cases build.**
  A pattern, an import path, a missing invocation target, an unreadable directory or an unparsable test file all keep the build, so a narrowing mistake costs time, never a test's binary.
- **Left out on purpose:** the study's other proposal, running `go vet ./...` in place of `go build ./... && go vet {targets}` in the hub's tier1 command.
  That is hub config, not code, and stays apart from this change.

## Tests

- `TestTestsUseLymBinary` (`internal/impactset`, tier0) covers external and in-package imports, a second file, another platform's constraint, a non-test importer, no test file, the kit's own directory, an unparsable file and a missing directory.
- `TestInvocationsUseLymBinary` (`internal/gatecli`, tier0) covers each target form: none, `./...`, a subtree pattern, an import path, relative and absolute directories, and a nested module's run directory.
- A new subtest of the gate scenario test in `internal/gatecli/test_integration_test.go` (tier1) runs a prebuilding tier on a package whose tests do not import the kit.
  It checks that the tier ran with no build and no exported variable.
  The existing prebuild subtests target a directory that does not exist, which still counts as using the binary, so they still build.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
