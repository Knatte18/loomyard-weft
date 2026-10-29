# Name failing tests in the adopt check mail and fix two flaky creel tests

The adopt check's mail for a failed tier now names each failing test with its first failure line and the `suite.log` path.
Before this, it gave only counts, so finding the failing tests meant reading `suite.log`.
Two flaky creel tests, the hook-wait restart subtest of `TestDaemon_Lifecycle` (#895) and the `TestCreelCLI` EXTERNAL row (#901), now synchronise on signals instead of counts that a loaded machine could satisfy early.

## Failed tests in the gate record and the adopt mail

- `internal/gatecli/counters.go`: the `go-json` counter keeps, per test in flight, the first output line that is not one of `go test`'s `=== ` or `--- ` status lines, trimmed.
  On a `fail` event it records the package, the test and that line.
  `Failures()` leaves out a test with a failed subtest in the same package, so it lists leaves only.
  The `vstest` counter returns none.
- `internal/gateslot/runlog.go`: `RunRecord` gains `failures` (`TestFailure`: package, test, first line), omitted when empty, which `tierrun.go` fills from the counter.
  `TierObserver.Ended` already hands the record over, so the adopt check needs no new seam.
- `internal/adoptcli/check.go`: `CheckTier` gains the same list, so `check.json` and `lym beam status`'s check view carry it.
  `tierNotice` renders a failed tier's mail as the unchanged summary line, then `<package> <test>: <first line>` per failure, up to `maxListedFailures` (10) followed by `and <n> more`, and then `log <suite.log path>`.
  The mail is now several lines long, and the summary stays its first line, the subject creel shows.
  A green tier's mail is unchanged.
- Docs: `internal/gatecli/doc.go` and `internal/adoptcli/doc.go` describe the new record field and notice lines.

## The hook-wait restart flake (#895)

The issue suspected that the first daemon released the hub late.
It does not: `creelengine.Run` releases its lock in a deferred call before returning, and the test stops a daemon by waiting for `Run` to return.
The extra daemon came from the test's own sync.
It counted idle reads to tell when the waiter had registered.
A read that the post caused could pass for that registration, so the test could stop the first daemon before the waiter connected.
The waiter's client then found no daemon, ran its spawn seam (`r.spawn`), and that spawned daemon took the hub from the test's own `r.start()`, failing with "another daemon holds this hub".

The subtest now starts two hook-waits for the same strand and waits for one of them to be answered `superseded`.
The daemon supersedes the older waiter only when the newer one registers, so that answer proves the survivor is registered with the first daemon.
The survivor is the waiter carried across the restart.
The waiters get no spawn seam, so no daemon can start behind the test.
The idle hook and the probe draining are gone.
After the restart, the order of the survivor's re-registration and the second post no longer matters: both orders end in a ring.

## The EXTERNAL row flake (#901)

Its TTL-expiry step took a snapshot of the probe-wait count, then advanced 61 s once the count grew by one.
Its removed-receiver step used `sweepStep`/`sweepLast`, which also work from counts.
Under load, a wait that an earlier verb's worker or the sweep recorded late satisfied the count.
The clock then moved before the verb's own work was queued, and the verb never returned.
Both steps now go through a new `advanceUntilReturned` helper over `creelfake.Clock.AdvanceByWaitsWhere`.
It advances the clock by each newly recorded probe or sweep wait until the verb returns.
The other row that uses `sweepStep`/`sweepLast` is unchanged.

## Design decisions

- The failure names come from the counter's event stream rather than from parsing `suite.log`, which holds every tier's text and would need per-tier offsets.
- A failing parent test is left out next to its failing subtest, because the parent's own failure carries no line of its own.
- The lifecycle fix changes only the test.
  The daemon's shutdown order was already correct, so the test needs neither a retry nor a wait for the release.
- Out of scope: the other count-based waits in the creel suites that no failure has pointed at.
  Also out of scope: a gate that tells a load-induced failure from a real one.

## Verification

- `lym gate test --wait` (tier0, diff scope) green after each commit.
  `lym gate test --tier 1 ./internal/creelcli ./internal/creelengine` green.
- Stress runs: the hook-wait restart subtest with `-count 200` and the EXTERNAL row with `-count 50`, all passing.
- New counter rows cover a failed subtest under a failed parent, a panic's first line and a failure with only status lines.
  The adopt check tests pin the exact failed-tier mail, the cap with `and <n> more`, and the failures in `check.json`.
