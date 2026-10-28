# Fixer report — round 1

## Changes

- F1 (`internal/battenshed/notice_test.go`): `TestNotice_UsageLimitOfOneRunOutranksTheOthersAndStandsWhileItReadsLimited` drives two runs: a limited run beside a plain API-error run, and beside a clean run, across repeated calls.
  Each asserts exactly one usage-limit notice and no API-error notice.
- F2 (`internal/shuttleengine/activity_test.go`): two rows, a run's own limited turn end carried into `AgentActivity.UsageLimit`, and a limited turn end before the prompt offset left zero.
- F3 (`internal/orchengine/state_test.go`): `TestResetForFreshLaunch` sets and asserts the three usage-limit fields cleared.
- F4 (`internal/orchengine/watcher_usagelimit_test.go`): `TestWatcher_UsageLimitWakeFailuresRetryTheWake` covers a wake line that does not render (wake moves a retry on, nothing sent) and a failing creel inject (wake stays due, next tick sends).
- F5 (`internal/orchengine/usagelimit.go`): a reset already past sets `UsageLimitWakeAt` to now; only a zero reset waits `usageLimitRetry`.
  Test `TestWatcher_UsageLimitReadAfterItsResetWakesTheOrchAtOnce`.
- F6 (`internal/shuttleengine/claudeengine/usagelimit.go`): a month and day the month lacks returns the zero time; a Feb 31 row in `TestUsageLimitResetsAt`.
- F7: the wake inject passes `stepInjectTTL`.
- Self-check follow-ups: `internal/orchengine/doc.go` and `internal/shuttleengine/claudeengine/doc.go` state the new wake and reset behavior; a limited-run-first row in the F1 test.

F4, F5 and F7 share one commit because they touch the same files.
`lym gate test` over `internal/battenshed`, `internal/orchengine` and `internal/shuttleengine/...` is green.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, MEDIUM: `orchengine/doc.go` wake timing stale after F5 — fixed as proposed.
- fix-consistency, LOW: `doc.go:374` step-inject sentence — no change needed, nothing contradicts it.
- fix-consistency, LOW: `claudeengine/doc.go` omits the F6 case — fixed.
- fix-affected-set, LOW: same `orchengine/doc.go` site — duplicate, fixed.
- fix-code-correctness, LOW: notice test lacked a limited-run-first row — fixed.
- fix-proposal: no findings.
