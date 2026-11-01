```yaml
lens: fix-way-forward
outcome: reported
```

Fix-way-forward lens: checked that each way forward the fix added can be carried out from the state it names and makes progress.

1. **MEDIUM**: D6's new bullet for an open fix task. The changed text is "for `paused`, the way forward of the pause reason its run record carries".
   - **Summary:** a paused run record's pause reason is a bare label and can be empty, so the notice can point the operator at a way forward that does not exist.
   - **Evidence:** `internal/shedengine/goto.go:133` writes the label `goto to <target>` (with who asked), and `goto.go:247` writes `re-launched onto <target>`.
   - At `internal/shedengine/run.go:893-895`, a routed pause carries whatever `routedState` returns.
   - At `internal/sheddriver/driverules.go:222`, an empty reason raises a friction note (`FrictionNote: stop.Reason == ""`), so an empty reason is a known case.
   - None of these reasons names a step. The notice should name a concrete resume step for a paused run instead of deferring to the reason. `internal/battencli/arm.go:586` resumes a paused state silently on a batten pass.
2. No further findings:
   - The drop path (`lym batten cancel <slug> --remove` then `lym board remove <slug>`) can be reached from a running or paused task, because the cancel has a run claim. D8 says the hand removal runs `adopt-check`.
   - For the empty-status case, "its landing starts the next check" holds, because dobby starts the task (D4 acceptance criterion).
   - The tries-file reset by a hand adopt and the new D6 acceptance rows can be reached.
   - `awaiting` now counts as halted for D6's removal. `cancel.go` and `cancelcmd.go` have no branch on the run state, so cancel does not refuse it.

Tool calls:
- grep of the state handling in `cancel.go` and `cancelcmd.go`, and of the `PauseReason` writers in `shedengine`: my context did not show whether cancel refuses `awaiting`, or what a pause reason contains.
- `sed` of `internal/shedengine/run.go:880-900`, and a grep for the files that handle `StatePaused`: to confirm how a routed pause reason is set and where a paused run is resumed.
