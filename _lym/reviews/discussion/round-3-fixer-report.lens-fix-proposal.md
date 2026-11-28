```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the F16 and F17 proposals the fixer carried out as written.

Proposals challenged:
- F16: delete the motif-run clear, its Bound clause, its criterion and its Constraints fact, and keep the batten refusal's `lym batten cleanup --apply` clause.
- F17: delete the Worktree-Create sentence along with the F16 deletion.

Findings:

1. **MEDIUM**: the clause F16 keeps in D5's batten bullet ("when the slug holds a done motif run, it first names `lym batten cleanup --apply` …").
   - **Summary:** the reviewer's reason for keeping the clause does not reach it. The batten check never runs over a done motif run, so the clause names a step that no refusal can show.
   - **The state the reviewer cites:** a task whose Sketch was refused stays a draft task over its done motif run. `sketchFinish` leaves that task "with its decision record stored" (`internal/loomcli/wiring.go:318`), so D5's refusal ("it holds none") does not fire for it.
   - **Where the check runs:** D5 runs batten's check "for a run or step with no status file". A done motif run leaves a status file. In `refuseAtStart` (`internal/battencli/startguard.go:530`), a done status with no clear keeps `found` true, and the guard refuses the start as done. The jacquard check is never consulted.
   - **What follows:** with the clear removed, the sentence "jacquard builds … other discussed tasks" does not hold for that refused-Sketch draft task. A jacquard start of it is refused by the done-run guard, and no D5 text or criterion names that refusal's way forward.
   - **Possible fixes:** drop the clause from the batten bullet. Or state that a done motif run under the slug is refused by the existing done-run guard before the jacquard check, and name that guard's way forward.

No finding against F17: deleting its sentence with the F16 passage leaves nothing that depends on it.

Tool calls:
- Read `internal/battencli/startguard.go` around `refuseAtStart` (lines 505–560), to check whether the jacquard check runs over a done status. My context showed `startClearsMotifRun` but not `refuseAtStart`'s handling of a done status.
