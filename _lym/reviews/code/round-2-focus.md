---
round: 2
focus:
  - "Does the done-run clear (`battencli.ClearDoneRun` in `internal/battencli/cleanup.go`, called from `clearDoneRuns` in `internal/dobbyengine/donerun.go`) remove the finished run's task branch locally and on origin, as the board entry's Goal asks, and does a branch failure leave the run's records so the next tick retries?"
  - "Does every git or remote-ref step the done-run clear takes go through `gitexec`/`gitrepo` and fabric under the remote-refs lock, as `PATTERN-gitrepo-client-boundary` and `PATTERN-fabric-boundary` require?"
  - "Do `internal/beamcli/doc.go` and `docs/overview.md` (the merge-parent and `lym batten cleanup` passages) match the dobby's done-run clear and the refusal text `doneSlugRefusal` in `internal/battencli/arm.go`?"
  - "In `internal/dobbyengine/donerun.go`, is a `CountedLandings` entry for a cleared slug dropped even when an earlier state update failed, and do the tests in `internal/dobbyengine/donerun_test.go` pin that the clear waits for the landing count?"
---

The focus targets the open findings' sites; it asks questions and caps nothing.
