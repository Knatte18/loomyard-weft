```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F6, F8–F13, and F3's carried-out part).

**Findings**

1. **MEDIUM**, `internal/reedengine/doc.go` (added lines on who clears a hold), with `stopForRelaunch` in `internal/shuttleengine/stop.go`. The new doc sentence says "a caller whose release failed removes the strand when it gives the run up". The code never does that on the path F3 describes.
   - When `EndStrandProcess` fails over a live strand and `ReleaseStrandHold` also fails, `stopForRelaunch` returns an error. `relaunchUnderAccount` wraps it as `errRelaunchDeclined`, and `switchAccountAtLimit` then handles the limit as without a switch (`accountrelaunch.go:148-151`). The run goes on, so nothing gives it up and no removal happens.
   - The strand stays live and held, and reed's resume skips it if it later dies. The fix only joins the release error into a Warn log, so F3's stated premise is unresolved.
   - The reviewer's "died path removes a held strand" proposal is also a no-op, because `strandRemovable` already includes `Held` (`stop.go:218`). The fixer left it out, but the doc now promises behaviour no code provides.

2. **LOW**, `readRegistry` in `internal/shuttleengine/claudeengine/resume.go` (F1 proposal: treat `fs.ErrNotExist` as an empty registry).
   - A config directory whose sessions directory is missing because the whole directory is gone, for example an unmounted or unavailable account volume, now reads as "no holder".
   - `checkResumable` then stays silent, `sessionHolder` finds nothing, and `processLiveness` can read dead, although a live process under that directory may still hold the session. That reopens the two-processes-one-session hole the warning existed to prevent.
   - Evidence: `resume.go` `readRegistry` now returns `nil, nil` on `ErrNotExist` for any registry path, not only the `sessions` child of an existing config directory. A narrower check, where only a missing `sessions` directory under an existing config directory counts as empty, would avoid it.

3. **LOW**, `TestRun_SwitchAccountAtLimit_ADeclinedRelaunchHandlesTheLimitAsWithoutASwitch` in `internal/shuttleengine/accountrelaunch_test.go` (F10 proposal: make the record write fail).
   - The row fails when the record write is made to fail with `permkit.Chmod(h.runDir, 0o555)`. Under root or on Windows, chmod does not block writes, so the relaunch proceeds and the test fails there.
   - The existing `unwritable_record` row in `stop_test.go` has the same exposure, so this matches precedent. It is still a test that passes or fails depending on the user the suite runs as.

No findings against F2, F4, F5, F6, F8, F9, F11, F12 or F13.
- **F4:** the seeding of `turnEnds` from `LaunchOffset` to the cursor holds, because `reseedAfterRelaunch` sets `LaunchOffset` equal to the cursor, which seeds 0.
- **F12:** the lock probe conflicts within one process under flock.

**Further tool calls:** none. My context already held every changed file.
