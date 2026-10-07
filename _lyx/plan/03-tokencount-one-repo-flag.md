# Card 3 — tokencount reads every fabric-repo fact through one flag

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/calibrate_integration_test.go`

**Create:**
- `tools/tokencount/main_test.go`

**Intent:**
Implement D8.
Remove the `history` flag.
The existing flag naming the prime's fabric repository becomes the one source for every read the tool makes: the archive tags of finished runs, the runs' plan commits and (cards 5 and 6) the runs' webster records.
Its default, the current worktree's sibling from `lyxcwd` and `fabricengine.WeftWorktree`, is now resolved whenever the flag is empty and either no slugs are named or the calibration is asked for, so `-calibrate <profile>` needs no other flag; `Calibrate` receives `gitrepo.New` of that one directory as its plan history.
Declare the flag's name as a non-test constant in `main.go` and use it in the `flag` call, because the fabric-vocabulary scan (`PATTERN-fabric-vocabulary`) covers every test file and forbids the side's name in any identifier, string literal or comment there: tests build the flag from that constant and never spell it.
The package comment drops the `history` paragraph and states that the one flag names the repository holding tags, plan commits and webster records.

Tests:
- `main_test.go` (untagged): `run` refuses the retired `history` flag with the flag package's undefined-flag error and writes no report.
- `calibrate_integration_test.go`: extend the existing test so it also drives `run` end to end with `-calibrate`, `-config`, `-hub`, `-projects`, one slug whose transcript the test writes into a temp projects directory, and the one repository flag (built from the constant) naming the test's own git fixture, and asserts the report holds the calibration section with the fixture's row.
  `run` reads the base trees from the process's working directory, so the test makes the fixture that directory with `t.Chdir`, which bars `t.Parallel`, and a comment on the test names that global state (`PATTERN-test-speed`).
  Comments and strings in both test files must not name the fabric side.

**ImpactSummary:** tokencount's flag set and its package comment change; no internal package is touched.
