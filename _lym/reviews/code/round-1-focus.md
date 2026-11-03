---
round: 1
focus:
  - "internal/dobbyengine/passes.go `planPass` and `drivable`: does a full pass on a paused run actually resume it through Land and Worktree-Teardown, and what happens when the pass ends with the run still paused (does each probe tick or status write re-launch a pass in a loop)? Check against the Goal's idea that dobby finishes such a run."
  - "internal/dobbyengine/passes.go `planPass` branches for `run.Held`, owned-elsewhere runs, `rs.FullPassOwed` and `OriginOwed`: does a paused, reached-parent run follow the same rules as a running one in each branch, and does the new `run.State != runStateRunning` clause change the continue-pass versus full-pass choice anywhere it should not?"
  - "internal/battencli/dobbyreads.go `DobbyRuns` new read: does `hubChildReachedParent(prime, slug)(run.status, true)` apply the same judgment `lym batten cancel` refuses on, including its second argument and its failure modes? Check the Warn-and-leave-unreached path, and that the read runs for paused runs only without extra cost or side effects for others."
  - "Hold semantics: the Goal says the pause came from an adopt hold or an orch pause. Does the diff respect a `lym tender pause` entry (covered) and the adopt hold, and is the orch-role.md claim that `lym batten pause` no longer holds such a run true of the code?"
  - "Docs landing with the change: contracts/orchrole/orch-role.md, internal/dobbyengine/doc.go and internal/beamcli/doc.go against the repository's own rules and the code as the diff leaves it; any other doc or spec naming 'a paused run gets no pass' left stale?"
  - "Test economy in internal/dobbyengine/passes_test.go: which new `TestPlanPass` rows duplicate another row's coverage, and is the `DobbyRuns` ReachedParent read (internal/battencli/dobbyreads.go) or the `drivePasses` `liftPending` change covered by any test?"
  - "Commit message and any change description: does each claim bear out against the diff?"
---

Seed focus for round 1; the Goal is the only answer key, with no plan.
