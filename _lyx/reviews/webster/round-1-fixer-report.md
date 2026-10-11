# Fixer-report — webster round 1

## Changes

- **F1** (BLOCKING, `1273b4764`): `internal/battenshed/innerrun_publishretry_test.go` now calls `t.Parallel()` first in the four harness-only top-level tests and in their subtests.
  `TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt` stays serial, with a header line saying `logcapture.Capture` replaces the process-global logger output.
  The harness state is per instance (`t.TempDir()`, its own `fakeClock`), so nothing is shared.
- **F2** (NIT, `12c6bace0`): `internal/battenshed/land.go` no longer reads the no-notice rule back out of the reason text.
  `stop` (notifies) and a new `pause` (no notice) share `stuck`, and `acquireLandLock` and `waitCleanPrime` return an explicit `paused` flag that `Call` routes to `pause`.
  `pausedReasonPrefix` is gone.
  This takes the finding's "split a `pause` method from `stop`" option; `stop` keeps its signature for its other call sites.

`lyx gate test ./internal/battenshed` passes.
The existing land tests cover the pause path (reason text and no notice), so no test was added.

## Disputed

None.

## Deferred

None.
