# Card 34 — goto-way-forward-address

**Edit:**
- `internal/shedengine/shed.go`
- `internal/shedengine/run.go`
- `internal/shedengine/goto.go`
- `internal/shedengine/step_test.go`
- `internal/shedengine/goto_test.go`
- `internal/shedbuild/newshed.go`
- `internal/shedbuild/newshed_test.go`
- `internal/shedverbs/spec.go`
- `internal/shedverbs/goto.go`
- `internal/shedverbs/goto_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/arm.go`
- `internal/battencli/wire.go`
- `internal/battencli/arm.go`
- `internal/battencli/specfill_test.go`

**Intent:**
Fix PR #321 findings 4c and 4e: two engine way-forward texts name a verb that does not work where the refusal is reached.

4c: the budget-exhausted reason and the missing-producer refusal in `run.go` name `lyx shed goto --to X` with no run-id, which addresses `self` and misses a slug-addressed run such as a batten lifecycle.
4e: `missingStatusWayForward` names `lyx shed seed`, which creates no status file for batten, whose own `run`/`step` creates it, so the named way forward loops.

Changes:

- `Shed` gains two told fields, `RunID string` (the addressed run-id) and `MissingStatusWayForward string` (the trailing `way forward:` clause for a missing status file).
  `GotoRequest` gains the same two fields.
  `shedbuild.ShedPaths` gains both and `NewShed` copies them onto the `Shed`, as it copies the other told fields.
- One unexported helper in `run.go` renders the goto command a text names: `lyx shed goto <run-id> --to <row>`, or the current run-id-less form when `RunID` is empty.
  The budget-exhausted reason, the missing-producer refusal and `Goto`'s own refusals that name a goto use it; the running refusal card 33 added names `lyx shed pause <run-id>` and `lyx shed step <run-id>` the same way.
  `ReasonBounceBudgetExhausted` stays the reason's exact prefix.
- `Step` and `Goto` use the told `MissingStatusWayForward` when it is non-empty and today's generic `missingStatusWayForward` otherwise.
- `shedverbs.Spec` gains `MissingStatusWayForward string`; the goto verb body passes it and `Spec.RunID` into `GotoRequest`.
- Loom fills `ShedPaths.RunID` with `c.runID` at both `ShedPaths` literals in `wiring.go`, and both `MissingStatusWayForward` fields with `way forward: run "lyx loom start" first to bootstrap this task`.
- Batten fills `ShedPaths.RunID` with the slug in `wire.go`, and both `MissingStatusWayForward` fields with `way forward: "lyx batten run <slug>" creates the lifecycle's status file`, the slug spelled out.

Tests:

- `step_test.go`: an exhausted budget and a missing producer on a `Shed` told `RunID: "some-slug"` each name `lyx shed goto some-slug --to`; a missing status file on a `Shed` told a `MissingStatusWayForward` carries that text and not `lyx shed seed`.
- `goto_test.go`: a missing status file with a told `MissingStatusWayForward` carries it.
- `newshed_test.go`: `NewShed` copies `RunID` and `MissingStatusWayForward`.
- `internal/shedverbs/goto_test.go`: the verb passes `Spec.RunID` and `Spec.MissingStatusWayForward` through, seen in a refusal envelope's message.
- `specfill_test.go`: batten's armed spec for goto carries the batten missing-status text naming the slug.

**ImpactSummary:** Two told fields added to Shed, ShedPaths, GotoRequest and Spec, filled by loom and batten; only refusal texts change.
