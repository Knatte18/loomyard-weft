# Build the prebuilt lym only when a test of the run may use it

`lym gate test` built `lym` on every run of a tier with `prebuilt_lym`, whether or not a test used the binary.
The study in `docs/research/speed/gate-fixed-cost-2026-11-27/report.md` timed that build at 2.4 to 2.8 s,
of about 6 s spent before the tests in a `tier1` call on one small package.
Such a run now builds only when a test it runs may use the binary, both with and without `--copies`.
A run that skips the build logs that it does, and exports no `LYM_PREBUILT_LYM`.

## When it still builds

The tier asks for it, and at least one of these holds:

- a `_test.go` file directly in a targeted directory imports `internal/testkit/lymbin`;
- the kit's own directory is targeted;
- a target holds `...` or is not a relative or absolute directory, so a run over the whole set `./...` builds;
- an invocation has no target;
- a targeted directory or test file cannot be read or parsed;
- the tier's command takes no targets, so it runs whole whatever the caller targeted.

## Design decisions

- `lymbin` is the one route by which a test gets the prebuilt binary, and the Testkit Invariant keeps every file
  other than a test file from importing it, so test files' direct imports decide the question.
- The new `impactset.TestsUseLymBinary` parses imports only and runs no `go list`, so the decision precedes any build.
- It ignores build constraints, and every uncertain case builds.

The `PrebuiltLym` and `PrebuiltLymEnv` docs, `pattern/PATTERN-test-speed.md` and the `gatecli` package doc state the condition.
`docs/benchmarks/running-tests.md` no longer compares the `cmd/testtiming` build to what `lym gate test --tags` does.

## To check by hand

- A tier whose command takes no targets runs whole, so `needsPrebuiltLym` (`internal/gatecli/prebuild.go`)
  always builds for it and never reads the caller's targets.
  Confirm that this is the wanted behaviour.
- The changed code was not timed, so the time saved is unmeasured.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
