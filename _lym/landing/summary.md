# Run reedengine tier2 tests in parallel; fold one battenrecipe duplicate

The first test-time audit flagged a 40 s `internal/reedengine` `tier2` cost and three packages whose test counts had grown far beyond their code.
This change acts on those leads, and corrects the audit where its readings were wrong.

## `internal/reedengine`: the executable lookup becomes an engine field

The audit's "2 tests, 40.00 s" was a reading of interleaved log lines, not a waited-out duration.
The package's `tier2` tests ran serially, each building its own tmux server, so their times summed to the package's wall time; the slowest was 11 s.
No fixed wait exists to inject.
The speed-up comes from running the tests in parallel, which `PATTERN-test-speed`'s "Tests run in parallel" already requires.

What held them serial was the package-level `executablePath` variable in `panebin.go`, which tests swapped to stand in for the test binary.
It is now an `Engine` field that `New` sets to `os.Executable`, and `composePaneLaunchLine`, `composeWindowCommand`, `composeSwitchCommand` and `composeFollowCommand` take the lookup as a parameter.
The callers in `spawn.go`, `window.go` and `bindings.go` pass `e.executablePath`.
Production behavior is unchanged: every engine `New` builds resolves the path through `os.Executable` as before.
An unresolvable path still logs its named `logger.Warn` and drops the prelude or the binding.

Every `tier2` test that touches no process-global state, and every `tier0` test that was serial only because of the seam, now calls `t.Parallel`.
Tests that use `t.Setenv` or capture the process logger stay serial.
The tests this diff rewrote name that state in a comment.
`doc.go` and the `panebin.go` header describe the new seam.

## Per-package test counts

Counts are `func Test` declarations on the branch against the audit's figures.

| Package | Audit, before to after | Source, before to after | Result |
| --- | --- | --- | --- |
| `internal/shell` | 11 to 65 | 11 to 14 | unchanged |
| `internal/boardengine/boardtest` | 3 to 68 | 3 to 3 | unchanged |
| `internal/battenrecipe` | 8 to 57 | 8 to 9, then 8 | one duplicate folded |

The audit's figures came from splitting interleaved gate logs per package and are not these packages' tests.
`internal/shell` holds 14 tests carrying 60 subtests, and `boardtest` holds 3 tests with 7 subtests; the audit's counts included such subtests.
`internal/shell` keeps one table per `Shell` method, each pinning a distinct method's output in both dialects.
`boardtest` keeps three distinct contracts.
Each test pins distinct behavior, so both packages are left as they are, as the board entry directs.

In `internal/battenrecipe`, `TestNameRunShed_ValueIsPinnedAgainstASymmetryRename` repeated the row-name loop of `TestNew_RowNamesMatchTheDurableIdentityConstants` over the same eight names.
Its one distinct assertion, that `NameRunShed` reads `Run-Shed`, moved into the row-name test with the rationale for the pin, and the duplicate was deleted.
No coverage is lost.

## Audit correction

`docs/benchmarks/test-suite-timing.md` gets a correction note under the audit's prune-task candidates, naming the real counts and the reedengine reading.
The audit's own numbers stay, since the section is a measurement report.

## To check by hand

- `internal/reedengine` has serial tests that capture the process logger without a comment naming that state, in files this diff did not rewrite, among them `attach_test.go`, `launchscript_cleanup_test.go`, `lifecycle_test.go`, `namerepair_test.go`, `probe_test.go`, `reconcile_test.go`, `windowsize_test.go` and `TestStageLaunchScript_WriteFailureDegrades` in `panebin_test.go`.
  The list is unverified; whether a package-wide comment sweep belongs in this change is open.
- Parallel `tier2` tests raise the load on the tmux servers they start, so a bounded wait could time out on a loaded host; run `lym gate test ./internal/reedengine --tier tier2` several times to check.
