```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every refusal, notice, next step and criterion the fix introduced can be carried out and leads to progress.

Ways forward the fix introduced:
- Decision 3: the wiring-failure line goes to `Geometry.LogPath`, and the failed-re-exec notice is posted after a re-exec.
- Decision 7: batten refuses the run when `machinename.Name` fails, with a way forward to fix the hostname.
- Decision 10: a check that finds the lock held exits 0, and `lym adopt` names the running check and spawns none.
- Decision 11: a running record whose start ticks don't match shows `ended without a verdict` with the rerun.
- Open risks: return to the template's `adopt` through `lym config gate`.
- Acceptance criteria rows added for each of the above.

Findings:

1. MEDIUM: Decision 3's new wiring-failure bullet ("When the state record holds this process's pid and start ticks … `runDobby` also posts the failed-re-exec notice to the nearest live orch").
   Summary: the notice can't be posted when the failing wiring step is `lymcwd.ResolveWorktree`, because the orch notice path needs the resolved worktree.
   Evidence: `internal/beamcli/dobbydeps.go:121` posts through `orchcli.NotifyFromScoped(worktree, …)`, and `:158` finds the orch through `hubgeom.NearestLiveOrch(worktree, …)`. `worktree` is the result of `ResolveWorktree` at `:71`, which is the first wiring step that can fail.
   The record should say the notice is posted only when the worktree resolves, and that otherwise the `dobby.log` line is the only trace.
   The matching Acceptance criterion should be bounded the same way.

2. LOW: the same bullet's notice "naming the error".
   Summary: after a wiring failure the dobby is gone, but the notice names only the error and no restart.
   Evidence: Decision 3 names `lym beam start` as the way forward only for a refused re-exec. The failed-exec notice gives only the error because "the dobby keeps running". A wiring failure leaves no dobby, so the orch gets no next step.
   The notice should also name `lym beam start` once the cause is fixed.

3. LOW: Decision 10's "A check that finds the lock held exits 0 without writing …", together with "The adopt envelope's `suite` names the check's pid".
   Summary: if a second `lym adopt` runs after a spawn but before the child takes the lock, it can still read the old record, spawn a check that loses the lock and exits, and name that dead pid in its envelope.
   Evidence: the fix's own Bound says "Between a spawn and the child's taking the lock, `lym beam status` can still show the previous record". In that window the running test can't see the first check, so the second envelope's pid points at a process that did nothing.
   The log and record paths in that envelope are still right.
   The record should say the envelope's pid may be a check that lost the lock, and that the record names the check that runs.

Tool calls:
- Grepped `internal/beamcli/dobbydeps.go` for the Notice and NearestOrch wiring, because my context showed only lines 55–100 of that file.
- Grepped `internal/gateslot/config.go` for `MigrateConfig` and `lym config gate` uses, to check whether a hub value equal to the template is pruned (which would defeat the Open risks way forward); my context lacked that code.
- Grepped `gateslot`'s prune and template wording for the same question. I found no template-equal prune, so there is no finding there.
