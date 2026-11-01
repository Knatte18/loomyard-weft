```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every code fact that the fix's new or rewritten sentences assert against the code at HEAD.

**[LOW] D4's new slot paragraph overstates what `slotsTaken` counts**

- **Text:** "dobby's `slotsTaken` counts every listed run not recorded owned elsewhere, held runs included".
- **Problem:** `slotsTaken` also leaves out any run whose `RunView` carries `NoSlot`, which is a pawl run whose landing hold does not stand. So "every listed run" is not quite true. The conclusion still holds, because held runs do take slots.
- **Evidence:** `internal/dobbyengine/starts.go:157-165` counts only runs with `!record.Runs[run.Slug].OwnedElsewhere && !run.NoSlot`. `internal/dobbyengine/passes.go:44` and `internal/dobbyengine/doc.go:108` define `NoSlot`.

These claims held against the code:
- The run-agent task guard reads a strand name that only `boardcli` sets: `cfg.StrandName` is assigned from the environment only at `internal/boardcli/cli.go:161`.
- `checkBeamPayload` is enforced on all three payload paths: `UpsertTask` (`store.go:507`), `UpsertTasksBatch` (`store.go:842`) and `MergeTasks` (`store.go:1021`). That covers `upsert`, `upsert-batch` and `merge`.
- `lym batten cancel` "clears the board claim" (writes the draft status) and keeps the pair unless `--remove` is set. None of its refusals stops a removal of a halted fix task started from the subject's worktree: the hold is slug-scoped, not a hub pause, and `refuseOwnPair` guards only the run's own pair.
- The `notify_on_land` memory never remembers a run first seen past its Land row (`internal/dobbyengine/landnotice.go:36`).
- Land's `PruneBoard` calls `Prune(true)`, which removes every done entry. `MaxSlugLength` is 32. The `light` profile sets darn `code_max_bounces` to 1. `max_runs` defaults to 3.
- A halted batten run state exists (`internal/battenshed/stophold.go`), so D7's `halted` state reads from something real.

Tool calls:
1. Grepped the `StrandName` setters and the `boardengine` merge and batch functions: my context held only the `checkBeamPayload` line numbers, not the function each sits in, and not whether anything besides `boardcli` sets the name.
2. Read `cancelcmd.go`'s help text and the `--remove` wiring, plus a grep for the halted state: my context had only `cancelcmd.go`'s grep hits, not the cancel refusals.
3. Grepped where the halted run state and `NoSlot` live: needed to check D7's halted source and the full `slotsTaken` condition.
