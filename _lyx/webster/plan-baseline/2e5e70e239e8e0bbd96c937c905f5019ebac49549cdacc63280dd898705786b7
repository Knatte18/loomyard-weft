# Card 15 — The parent-lock wait's pause-flag read failure logs once per episode

**Create:**
- `plan:internal/landingshed#TestParentGuard_PauseReadFailureLogsOncePerEpisode` -> `func TestParentGuard_PauseReadFailureLogsOncePerEpisode(t *testing.T)`

**Edit:**
- `internal/landingshed#parentGuard.pauseRequested` -> `func (g parentGuard) pauseRequested(failures *logger.Episode) bool`
- `internal/landingshed#parentGuard.ensure`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D11 for `internal/landingshed`: the site the audit in `00-overview.md` lists for this package as guard.

`ensure` passes `pauseRequested` as `parentlock.Waiter.Paused`, which `parentlock.Store.Acquire` calls on every poll of a wait with no time limit, and `pauseRequested` logged `landingshed: could not read the pause flag during the parent lock wait` at Warn on every poll while the read failed.
`parentGuard` is a value type, so the episode lives in `ensure`, not on the guard: `ensure` holds a local `var pauseFailures logger.Episode` for the length of its `Acquire` call and sets `Paused` to a closure calling `g.pauseRequested(&pauseFailures)`.
`pauseRequested` takes the episode: a read error calls `failures.Fail` with the existing message and `cause` field and still reads as not paused, and a read without an error calls `failures.Recover("landingshed: the pause flag reads again during the parent lock wait")`.
Its doc comment's "logs one Warn" becomes the episode rule.
No wait, pause or halt decision changes.

`TestParentGuard_PauseReadFailureLogsOncePerEpisode`, in `parentguard_test.go`, is serial (it captures the process-global logger with `logcapture.CaptureVerbose` plus `logger.SetVerbosity(2)`).
It builds the guard with `newGuardFixture`, sets `g.lock.PauseRequested` to a func that errors on its first three calls and then answers false, and calls `g.pauseRequested(&failures)` four times on one `logger.Episode`.
It asserts every call returns false, exactly one `level=WARN` and two `level=DEBUG` `could not read the pause flag during the parent lock wait` lines, and one `level=INFO` `the pause flag reads again during the parent lock wait` line.
`TestParentGuard_EnsureReasonsAndErrors`' pause row keeps passing unchanged, since `ensure` still halts on a pause.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/landingshed` row rises from 85 to 86.

**ImpactSummary:** one unexported method re-signed, with its one caller listed here, in `internal/landingshed`; no parent lock wait or pause decision changes.

**Commit:** 15: landingshed: log a repeating pause-flag read failure once per episode
