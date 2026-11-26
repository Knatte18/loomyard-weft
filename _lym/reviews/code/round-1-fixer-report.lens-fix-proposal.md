```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that were carried out as proposed.

1. **LOW**: F6 change, `waitReplacingInterrupted` / `fixerOutcome.replaceErr` in `internal/gatefixer/rowgate.go`. A replacement spawn error is still dropped silently when the judging run passes or is pending.
   - The only reader of `replaceErr` is `note()`.
   - `waitThenRun` returns at `result.Passed || result.Pending || result.Terminal` before it calls `note()`.
   - Case: a restarted driver finds a left-live fixer that ended interrupted, the replacement's `SpawnOrAttach` fails, and the inner gate passes (or fails terminally). The error appears nowhere.
   - F6's premise was "neither logged nor put into the findings", and the proposal covers only the findings.
   - The new test row `AFailedReplacementSpawnLeavesTheInterruptionStandingWithItsError` uses only a failing gate.

I built no concrete breakage for the other findings fixed as proposed (F1, F2, F3, F5, F8, F9, F10, F11, F12).

Outside my lens and not confirmed: `internal/shuttleengine/stepstop.go:158` also stops runs with `interrupted=true`. If such a step stop can kill a fixer while its driver is still waiting, the F2 change lets `Final` spawn a replacement, as the fixer entry's `Gate` already could.

Tool calls:
- One `grep` for `stopRecorded(` callers and for stale "final never spawns" or "run error degrades" statements. My context lacked the other writers of `interrupted` and the docs outside the files read. It found no stale statement.
