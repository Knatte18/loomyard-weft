```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix added or rewrote against the code at HEAD.

no findings

- **F1, the Constraints fact:** holds.
  - `agentActivityFor` (`internal/loomcli/looparm.go`) reads only a verify marker, a parent-lock wait, a drift lock, non-held wait markers and the activity of live agent runs.
  - Wait markers are written only by `showWait` (`internal/shuttleengine/waitmark.go`), which only gate, shell and held waits inside a run's wait loop call. A `StartSession` session never enters that loop, so it writes no marker.
  - `prreview.go:385-440` and `pawlengine/beamawait.go` hold no trace or logger lines.
  - `reviewWaitingFor` is used only as the `Waiting` hook (`loomcli/arm.go:419`, `checklist.go:136`), and `shedverbs` reads that hook only for the status and watch line (`status.go:100-124`, `209`).
- **F2, the stamp ("a halt and a resume write no such entry"):** holds.
  - `run.go:615` says a resume writes no entry.
  - `shedengine.Relaunch`, which does write an `OutcomeGoto` entry, is called only from the tackler and arbiter consume paths (`sheddriver/tacklerconsume.go:56`, `shedarbiter/consume.go:59`). When the target is already in effect it writes nothing (`goto.go:190-194`).
- **F3, "as PR-Review's wake does before its deciding read":** holds. `prreview.go:452-466` stops the session before `readTree` and the judge, and resumes it when the review goes on. It stops only when `sessionStopWanted` says the read may end the review, which matches a flag carrying the current stamp.
- **F5, "committed in its own diff":** matches the caller rule's package doc, `decision.go:11`: "in the call's one committed diff".
- **F9, F10:** design statements with no claim about the code to check. The cited Run-Shed text sits at `battencli/arm.go:514`.

Tool calls:
1. `sessionStopWanted`, the `arm.go` Waiting hook and the `waitmark.go` writer: my context had no stop condition, hook user or marker writer.
2. The `shedverbs` Waiting readers and the `showWait` callers: to confirm the status-only reading and which code writes the marker.
3. The `waitmark.go` function headers and the `waitingNote` callers: to see which run events show a wait.
4. Trace and logger lines in `beamawait.go` and `prreview.go`: to check "no trace line".
5. `Relaunch` in `goto.go`: to check whether a resume writes a goto entry.
6. The `Relaunch` request doc, its goto condition and its callers: same reason.
