```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each changed function and branch on concrete inputs.

**Per function or branch, the case I ran, and whether it is correct:**
- `loomengine.ClassifyHalt` with state `failed`, transient set, buildSkew and interrupted unset returns `transient-stop`. Correct.
- Same with interrupted also set: the interrupted case is earlier in the switch, so it returns `interrupted-stop`. Correct.
- Same with buildSkew also set returns `build-skew`, because the build-skew case comes before the transient case. Correct.
- With state `running` and transient set, it returns no kind. Correct.
- With state `blocked` and transient set, it returns an escalation, because blocked kinds come before any failed case. Correct.
- `loomCLI.failedHalt` on a producer error marked `github-api`:
  - shed runs `MarkTransient(s.Transient(err), err)` (`internal/shedengine/run.go:402-403`). An empty class leaves an already-marked error unchanged.
  - It persists `failed` with `TransientOf(callErr)` (`run.go:406`).
  - `transientError.Error()` passes the text through, so `st.Error == err.Error()` holds.
  - `st.Transient` reads `github-api`, so the note gets `Transient: true`.
  - Correct.
- `failedHalt` on an unmarked error: `st.Transient` is empty, because it is only written on a failed persist (`internal/shedengine/status.go:193`). It falls through to `producer-hard-failure`. Correct.
- `failedHalt` when the persist fails: the status does not read `failed`, so no note is written. This is the old behaviour and the diff does not change it. Correct.
- `failedHalt` on a stale `failed` status from an earlier step: the `st.Error` match blocks it, and the old status's transient class is never read for the new error. Correct.
- `parentGuard.ensure`, first `OnWait`: `waiting` is false, so it sets `waitStarted` and logs the start with the holder. Correct.
- `parentGuard.ensure`, a later `OnWait`: `Acquire` calls it only when the holder changes (`internal/parentlock/acquire.go:146-152`), so the "changed holder" line is accurate. Correct.
- `parentGuard.ensure` on every exit after a wait:
  - It logs the end line on grant, on `ErrPaused`, on `ErrUnreadable`, on context cancellation and on any other error, each with `granted` set to `err == nil`.
  - The line comes before the wait record removal and before the switch.
  - Correct.
- `parentGuard.ensure` when the lock is granted at once or the first `tryAcquire` fails: `OnWait` never runs, so no start or end line is written. That is consistent, since there was no wait. Correct.
- The wall-clock and awake arithmetic: `Round(0)` strips the monotonic reading, so `waited` is the wall-clock difference and `awake` is the monotonic difference.
  - On Linux the monotonic clock excludes suspend, so a gap shows a suspend. Correct.
  - On Windows it does not hold; see finding 1.
- `writeHaltNote` passes `Transient` through to `ClassifyHalt`. Correct.

**Findings**

1. LOW, design — `internal/landingshed/parentguard.go:9-10` (the comment on `ensure`).
   - The comment says the gap between the wall-clock and awake times "shows the host was suspended".
   - That rests on Go's monotonic clock stopping during suspend. It does on Linux.
   - I believe Go's Windows monotonic clock keeps counting through sleep, so on Windows the gap would be zero even after a suspend. I recalled this and could not check it from here.
   - The repository builds for Windows: `internal/proc/proc_windows.go`, `internal/fslink/fslink_windows.go` and other `_windows.go` files.
   - A wall-clock step, such as an NTP correction, also makes a gap without any suspend.
   - Evidence: `parentguard.go:36-40` in the diff.
   - Fix: limit the claim to platforms whose monotonic clock pauses in suspend, or make it a weaker hint.

2. LOW, consistency — `internal/loomengine/anomaly.go:34-36` and `internal/loomcli/doc.go:16-17`.
   - The comments say every transient-classed failed halt is "a network or host failure that the same step, run again, can clear" and that "the driver re-steps such a stop".
   - `ClassifyHalt` and `failedHalt` apply the kind to every non-empty class, including `agent-start`, an agent session that never became ready.
   - For `agent-start` and `host-resources`, `networkFailure` returns false (`internal/sheddriver/driverules.go:307-316`). So `decideDrive` gives the stop to the tackler under "a transient error that outlived its re-step" (`driverules.go:291-292`). The driver never re-steps it.
   - The only re-step is the loop's one immediate re-step (`internal/shedverbs/loop.go:463-466`).
   - So the comments misstate what happens to two of the four classes. The reflection template drops these halts on the same premise, which is outside this lens.

Tool calls:
- One `ls`/`grep` for `_windows.go` and `go:build windows` files plus `go version`, to learn which platforms the repository builds for. My context did not show that.
