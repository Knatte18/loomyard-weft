# Gate: {test_targets} starts only the test binaries of the tier

A tier command can now hold `{test_targets}`, which `lym gate test` fills with the targets narrowed to the packages that hold a test of that tier.
`go test` then compiles, links and starts no binary that would find nothing to run.
A whole tier 1 pass starts 196 test binaries, and 130 of them hold no tier 1 test.
On neve, starting only the 66 with one takes the floor from 20.5 s to 12.4 s with no test lost.
The saving should weigh more on OSL-1046, where Cortex XDR holds each binary it has not seen; that is not measured.

## Changes

- `impactset.HoldsTierTest` reads the `_test.go` files of a package directory and reports whether one declares a test function the tier runs.
  A function opening with `tiergate.Require(t, N)` counts for tier N alone, and any other for every tier.
  The list is derived, so a package gaining its first test of a tier enters without a hand edit.
- `gatecli` narrows each invocation's targets to the packages that hold a test of the tier or of a tier it `covers`, then renders them into `{test_targets}`.
  A pattern target is listed with `go list` first, and an import path is kept.
- An invocation left with no package runs no command, its other steps included, and says so on stderr.
- `gateslot` adds `TestTargetsPlaceholder` and `HoldsTargets`.
  `all_targets`, `{args}` checks for the guard step, and the "command ignores targets" test accept `{test_targets}` wherever they accept `{targets}`.
- The `gate.yaml` template, `gatecli/doc.go` and `pattern/PATTERN-test-speed.md` describe the placeholder.

## Design decisions

- `{test_targets}` is opt-in.
  `lym` cannot rewrite a hub's `go test` command, so no tier changes until its hub swaps the placeholder in.
  The example in `gatecli/doc.go` shows this hub's tier0 to tier2 as they read after the switch.
- Narrowing only saves time, so it fails open.
  The targets run unnarrowed when they cannot be listed or the narrowed list passes `greenCommandBudget`.
  A directory or test file that cannot be read or parsed counts as holding a test.
- Build constraints are not read, so a file only another platform compiles still counts.
- `TestMain` does not count as a test; `Test`, `Benchmark`, `Fuzz` and `Example` functions do.

## To check or run by hand

- Not run: the hub step that switches this hub's tiers.
  Run `lym config gate --set` for tier0, tier1 and tier2, replacing `{targets}` with `{test_targets}` in each command.
- Not run: one gate pass on OSL-1046 with `lym gate report`'s launch times, to see whether Cortex XDR holds the binaries.
- Check `runInvocations` in `internal/gatecli/tierrun.go`: its `{test_targets}` wiring (narrowing, the empty-invocation skip and the unnarrowed fallback) is only partly covered by tests.
