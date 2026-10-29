# Name failing tests in the adopt mail; make two creel tests deterministic

The adopt check's failed-tier mail now names each failing test with its first failure line and the `suite.log` path.
Before, the mail carried counts only, which left an earlier flake undiagnosable.
Two creel tests that failed under machine load are made deterministic.

Closes #895
Closes #901

## Adopt check mail

- The `go-json` gate counter in `internal/gatecli` records each failed leaf test as package, test name and first output line.
  The first line is the first one that is not a `go test` `=== ` or `--- ` status line, trimmed.
  A failed parent whose subtest also failed is left out, so the list names leaves.
  The `vstest` counter records none.
- `gateslot.RunRecord` gains an optional `failures` list, omitted when empty.
  `gatecli.TierObserver.Ended` already receives the run record, so the adopt check gets the failures with no new seam.
  The key is optional, so older readers still parse the record.
- `adoptcli.CheckTier` carries the same list, so `check.json` and the check view hold the names.
- A failed tier's mail keeps its summary as the first line, then one `<package> <test>: <first line>` line per failure, then the `suite.log` path.
  At most ten failures are listed, and a longer list ends with an `and <n> more` line; the log holds the rest.
  A green tier's mail is unchanged.
- The `doc.go` files of `gatecli` and `adoptcli` describe the new record key and mail layout.

## Flaky tests (test code only)

- `TestDaemon_Lifecycle` hook-wait restart subtest in `internal/creelengine/server_test.go` (#895).
  The daemon's shutdown path is not at fault: `creelengine.Run` releases its lock in a deferred call before it returns.
  The old sync counted idle reads, and a read not caused by the waiter's registration let the test stop the first daemon before the waiter connected.
  The waiter's own spawn then took the hub, and the test's restart failed with "another daemon holds this hub".
  The subtest now starts two hook-waits of the same strand and waits for one to end superseded, which proves the other is registered.
  The waiting clients get no spawn seam, so no daemon can start behind the test.
  The test no longer checks the idle read order of the registration; `TestScheduler` covers that.
- `TestCreelCLI` EXTERNAL row in `internal/creelcli/cli_test.go` (#901).
  The TTL-expiry and removed-receiver steps advanced the daemon clock once after a wait count taken from a snapshot.
  Earlier workers still recording waits could satisfy that count on a loaded machine, so the advance came before the verb's own work was queued.
  The steps now advance the clock by each newly recorded probe or sweep wait until the verb returns, through `creelfake.Clock.AdvanceByWaitsWhere`.

## For the reviewer

- The cap of ten listed failures is a judgment call.
- The EXTERNAL row's stepping also advances the sweep and chase timers alongside the probes.
  A later change that makes a sweep or chase act on alpha in that window could change the row's outcome.
- No new real-time waits: the new syncs wait on signals or the fake clock.
- Not changed: a gate that tells a load-induced failure from a real one, and other count-based waits in the creel suites that no failure has pointed at.
- Not run by hand: neither flake reproduces without load, so determinism under load was not exercised.
