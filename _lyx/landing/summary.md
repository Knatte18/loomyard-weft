# Prune six test suites and teach testtiming to judge them

The tests of `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli` and `internal/shedadapters` shrink without a pinned behavior lost.
Before, most of their tests sat in the redundancy report's "no coverage" list, outside the tool's judgment, so the tool is changed first and the suites pruned with it.

## Tool changes (`cmd/testtiming`)

- **Narrower exclusion.**
  A test is excluded from judgment only when it may run module code in another process: it references `internal/testkit/lyxbin`, re-executes its own binary (`os.Executable`, `os.Args[0]`), invokes the `go` toolchain, or lives in a `tmux`-tier file.
  Spawning git or another external binary no longer excludes a test.
  A test in an untagged file is never excluded as spawning or unresolved.
  A test in a tagged file with a call the scan cannot resolve stays excluded.
- **Keep directive.**
  `//testtiming:keep <reason>` directly above a `func Test…` line keeps a candidate.
  The report lists it under "Kept" with its reason.
  An empty or whitespace-only reason, a duplicate directive on one test, or a directive not directly above a top-level `func Test…` exits non-zero naming file and line.
  The directive changes report output only, and a kept test stays in the covering pool.
- **Section splice.**
  A `-redundancy` run with a `-pkg` narrower than the module replaces only the measured packages' sections of an existing `-out` file and leaves the header and other sections alone.
  A package with no tests loses its section, a package with no section gets one in import-path order, and a missing file is written whole.
  A tag set differing from the existing header's exits non-zero and writes nothing.
  A package that fails to list or build keeps its old section, the other sections are still written, and the run exits non-zero.
- The single-test run form is fixed so no test of the six packages is listed as "failed".

`pattern/PATTERN-test-economy.md`, `docs/benchmarks/running-tests.md` and the package doc describe the new rules.
`docs/benchmarks/test-redundancy.md` is regenerated for the six packages; every one now reads "No candidates".

## Test changes

- Redundant tests are deleted only from the removable set and after a read of their assertions; others are folded into table-driven tests with each former test a named `t.Run` subtest, or kept with a reason.
- Families of tests that each built the same hub, pair or tmux server became one scenario test that builds the fixture once and runs named steps in order.
  Scenarios stay within one tier, and a step that builds on an earlier one says so in a comment.
- Tests that needed no real git, tmux or process moved to the untagged tier.
  `internal/websterengine` gained a `Git` interface (`git.go`, carried by `Geometry`, nil meaning the real implementation) so its bracket verbs and run-level checks run against a fake in `gitfake_test.go`; tests that need real git sit in `*_git_test.go` files.
  Production passes today's git implementation, so behavior is unchanged.
- Skill-loading tests and the stub driver scripts in `internal/loomcli` are untouched; they belong to the parallel `skills-in-one-turn` task.

## Measurement

Full tables, run-by-run wall times and load averages are in `docs/benchmarks/test-suite-prune-core.md`.
Medians of three runs, with `-tags integration,tmux`, before and after:

| Package | Tests | Wall (s) | Coverage |
|---|---|---|---|
| `internal/fabricengine` | 698 → 610 | 43.59 → 29.27 | 81.9% → 81.9% |
| `internal/websterengine` | 456 → 175 | 10.66 → 2.48 | 86.2% → 86.3% |
| `internal/reedengine` | 330 → 212 | 24.73 → 18.96 | 82.4% → 82.4% |
| `internal/shuttleengine` | 218 → 129 | 0.47 → 0.43 | 90.6% → 90.6% |
| `internal/loomcli` | 278 → 167 | 89.00 → 77.30 | 63.4% → 63.4% |
| `internal/shedadapters` | 261 → 151 | 1.19 → 0.86 | 91.5% → 91.6% |

No package's wall time rose and no covered block became uncovered.
The before runs started at higher load (1-minute average 2.58, 7.25, 11.60) than the after runs (1.78, 3.45, 2.18), so part of each wall delta is quieter load; serial time carries less of it.

## To check

- Each deletion and each hook-test keep against the rule that a candidate which makes git run module code through a hook, alias or helper must be kept; the scan cannot see this, and the `internal/loomcli` and `internal/fabricengine` pass did not check every one.
- The keep reasons in `internal/reedengine` and `internal/loomcli` end in a generic clause not checked against every covering test.
- Timing run 1 after the prune reported `internal/reedengine` as FAIL under full-suite contention, with the test name not captured; it passed in two later runs and in `-count=3`, and is undiagnosed.
- `gofmt -l` flags `internal/fabricengine/pull.go`, which this change does not touch.
- The full gate was not run at plan level: `go vet` under each tag, `go test ./...`, and `-tags integration` and `-tags tmux` on the six packages, `cmd/lyx` and `cmd/testtiming`.
- `test-suite-prune-rest` runs in parallel and may touch `cmd/testtiming` or the report header.
- `internal/fabricengine/mergeerrors.go` changed only in a comment naming the surviving test.
