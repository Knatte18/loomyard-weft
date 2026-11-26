---
round: 2
exclude_lenses: []
focus:
  - "Watch memory and owed lines: where does the record keep them, and what deletes them? Trace a done run tick by tick through `internal/dobbyengine/passes.go:184-188` and `:319-340`, the step order at `internal/dobbyengine/loop.go:708-729` and the done-run clear, and settle whether each line the Goal promises is posted once, retried and dropped as decided, and which run list the watch reads."
  - "Overlap with the landing notice: for a landed run and a cancelled run, with `land_notice` on and off, which lines does the orch get? Check any skip against when `PastLand` turns true (`internal/battenrecipe/land.go:27-42`), when the notice memory is decided and dropped (`internal/dobbyengine/landnotice.go:119-177`) and when the derived state reads landed (`internal/runfsm/state.go:71-76`)."
  - "Cancel visibility: a cancel that removes its records deletes the batten status inside the command (`internal/battencli/cancel.go:544-565`), and `dobbyParentRuns` lists only slugs with a batten status (`internal/battencli/dobbyreads.go:104-111`). Settle whether the Goal, Scope, Decisions and acceptance criteria agree on what is reported for such a run and by whom."
  - "Stop predicate: walk these readings through the stops and level rules and settle whether each has one stated answer: a run with no child status yet (`internal/dobbyengine/runnotice.go:107-108`), a batten pause lifting over a blocked child, a child going from blocked to failed, a first `run_watch` read that fails, and a tick whose scheduling steps do not run."
  - "Gate-hold notice: settle dedup for a hold spanning several slots (`internal/gateslot/pool.go:179-201`), behaviour when `Pool.Holders` fails (`internal/gateslot/pool.go:353-362`), the accepted range of `gate_hold_factor` and `gate_hold_min`, and whether each fact the clock choice rests on is pinned."
---
