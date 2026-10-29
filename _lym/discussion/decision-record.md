# Decision record — creel-flakes-and-adopt-names

## Goal

The adopt check's per-tier failure mail names each failing test with its first failure line and the `suite.log` path, and the two flaky creel tests, the `TestDaemon_Lifecycle` hook-wait restart subtest and the `TestCreelCLI` EXTERNAL row, pass deterministically under machine load.

## Scope

- In: the `go-json` gate counter collects failed tests, the tier run record carries them, `CheckTier` keeps them and the adopt check's failed-tier mail lists them.
- In: the hook-wait restart subtest in `internal/creelengine/server_test.go` and the EXTERNAL row in `internal/creelcli/cli_test.go`, test code only.
- In: the docs of `internal/gatecli`, `internal/gateslot` and `internal/adoptcli` that describe the run record, the counters and the check's notices.
- Out: the creel daemon's production shutdown path, which already releases its lock before `Run` returns.
- Out: other count-based waits in the creel test suites that no failure has pointed at.
- Out: a gate that tells a load-induced failure from a real one.

## Decisions

### D1 — The go-json counter records each failed leaf test with its first line

The counter keeps, per test in flight, the first output line that is not one of `go test`'s own `=== ` or `--- ` status lines, trimmed, and on the test's `fail` event appends a failure holding the package, the test name and that line.
A failed test whose subtest also failed is left out, so the list names leaves.
The `vstest` counter records none.
Rationale: the counter already reads every event of the tier, so the names come at no extra pass over `suite.log`, and the first non-status line is where `t.Errorf`, `t.Fatalf` and a panic print the cause.

### D2 — The tier run record carries the failures

`gateslot.RunRecord` gains a `failures` list, omitted when empty, of package, test and first line.
`gatecli.TierObserver.Ended` therefore hands them to the adopt check with no new seam.
Rationale: the observer already receives the run record; the record is tolerant JSON, so an older reader skips the new key.

### D3 — The adopt check keeps the failures and lists them in the failed-tier mail

`CheckTier` gains the same failures list, so `check.json` and the check view hold the names.
A failed tier's mail keeps its summary as the first line, then one line per failure, `<package> <test>: <first line>`, at most ten with a closing `and <n> more` line, then the `suite.log` path.
A green tier's mail is unchanged.
Rationale: the operator reads the mail; the first line stays the subject creel shows, and the cap keeps a broken build's mail short while the log holds the rest.

### D4 — The hook-wait restart subtest proves registration through the supersede rule

The subtest starts two hook-waits of the same strand on the first daemon and waits for one to end superseded, which proves the other is registered; that other is the waiter carried across the restart.
The waiting clients get no spawn seam, so no daemon can start behind the test.
Rationale: the old sync counted idle reads, and a read not caused by the waiter's registration let the test stop the first daemon before the waiter connected, whereupon the waiter's spawn took the hub and the test's own restart failed with `ErrDaemonRunning`.
Bound: the test no longer checks the idle read order of the registration, which `TestScheduler` covers.

### D5 — The EXTERNAL row steps the daemon clock until the waiting verb returns

The TTL-expiry step and the removed-receiver step advance the daemon clock by each newly recorded probe or sweep wait until the verb returns, through `creelfake.Clock.AdvanceByWaitsWhere`, instead of advancing once after a wait count taken from a snapshot.
Rationale: a snapshot count races the waits earlier verbs' workers and the sweep still record, and on a loaded machine a stale wait satisfied it, so the single advance came before the verb's own work was queued.

## Constraints

- `PATTERN-test-speed`: no new real-time wait; the new syncs wait on signals or the fake clock, and each test change stays in its tier.
- `PATTERN-tolerant-records`: the new record keys are optional and an unknown key still reads.
- `PATTERN-test-isolation`: the creel tests keep their in-process daemons over `lymtemp` directories.

## Auto-mode assumptions

- The cap of ten listed failures is a self-pick; the log holds the full list.
- The issue's daemon-release suspicion does not hold: `creelengine.Run` releases its lock in a deferred call before it returns, and the test stops a daemon by waiting for `Run` to return, so the hub held at the restart was a second daemon the waiter's spawn started.

## Open risks

- The EXTERNAL row's stepping advances the sweep and the chase timers alongside the probes; a later change that makes a sweep or chase act on alpha in that window could change the row's outcome.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - run-record-failures-key-unpinned (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a counter row with a failing test whose output holds a status line and an error line, a failing parent with a failing subtest, and a panic reports exactly the leaf failures with their first non-status lines; a `vstest` tier reports none.
- D2: the tier run record of a failed `go-json` tier holds the failures, and a passing tier's record carries no `failures` key.
- D3: a failed tier's adopt mail lists each failing test with its first line and the `suite.log` path, a list past the cap ends with `and <n> more`, and `check.json` holds the failures; green-tier mail is unchanged.
- D4: the hook-wait restart subtest passes with no idle-read counting and no spawn seam on its waiters.
- D5: the EXTERNAL row's TTL-expiry and removed-receiver steps pass with no snapshot wait count.
