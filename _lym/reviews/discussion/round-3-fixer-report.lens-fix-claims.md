```yaml
lens: fix-claims
outcome: reported
```

Checked every claim the fix added or rewrote against the code at HEAD. One claim does not hold.

**MEDIUM: the way forward for a `paused` fix task is not in the run record**
- **Changed text:** the new D6 bullet says that for `paused` the notice names "the way forward of the pause reason its run record carries".
- **Problem:** a paused run record's `PauseReason` is a label, not a way forward, and it can be empty. The notice would point the operator at a step it does not have.
- **Evidence:**
  - `internal/shedengine/goto.go:133` sets it to `labelled("goto to "+req.Target, req.By)`.
  - `internal/shedengine/goto.go:247` sets it to `"re-launched onto " + req.Target`.
  - `internal/shedengine/run.go:893` clears it to `""`.
  - `internal/sheddriver/driverules.go:222` writes a friction note when `stop.Reason == ""` on a paused stop.

These claims hold:
- **Task-branch naming:** the task branch is fabric's branch prefix followed by the slug (`internal/battencli/wire.go:319`).
- **Cleanup leaves the branch:** a torn-down pair keeps its task branch locally and on the remote after `lym batten cleanup --apply` (the `doneSlugRefusal` doc comment, `internal/battencli/arm.go:860-864`, and `internal/battencli/cleanup.go:5`).
- **`awaiting` reads as halted:** `awaiting` behaves like blocked wherever a halt is observed (`internal/shedengine/status.go`).
- **Tries file is safe from the check record:** the check record's writes cannot touch a separate tries file, because `runCheck` writes only `check.json`, `suite.log` and the check lock.

Tool calls:
1. Read `internal/battencli/wire.go:315-336` and grepped where `PauseReason` is set, to check the task-branch naming and what a pause reason holds. My context had only the branch-prefix comment line.
2. Grepped `StatePaused` in battencli, dobbyengine and loomcli, to see how a paused run state is handled. My context had only the driver's hand-back rule.
