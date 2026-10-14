# Twill: spawn lens forks after the files are written

The twill writer, under both `discussion_producer: twill` and `plan_producer: twill`, now spawns its lens forks only after it has written the reviewed files and its self-check has passed.
Before this change it spawned them before writing, and they waited on a ready marker.
The forks now review the files as their inherited context holds them, without reading them again, and wait on nothing.
The benches in `docs/research/twill-fork-timing-bench.md` and `docs/research/twill-fork-timing-plan-bench.md` found this cheaper at the same quality: $4.44 against $5.43 per Discussion run, and $0.94 against $1.60 per Plan run for the lens forks.

## What changes

- **Directive** (`contracts/stencils/twill/twill-directive.md`): the procedure starts at reading the rubric, then spawns, waits, judges, records and runs the final self-check.
  Writing the files and the first self-check now happen before the procedure, and each host's spawn point says so.
  The fork boilerplate drops the `lyx shuttle await` loop and its give-up rule, and tells the fork the writer wrote and checked the files before spawning it, so it never reads them again.
  The 6-call cap and the report line for each extra read stay.
- **Relaunch rules**: a writer that died before its Agent call still spawns once, after it finishes writing and passes the self-check.
  A writer that died at or after its Agent call never spawns again, and records the lost forks as "no report", as before.
- **Spawn points** (`internal/loomengine/twill.go`): Discussion runs the procedure once both output files are written per Step 5 and `validate-discussion` passes per Step 6.
  Plan runs it once the plan is written per Steps 3 and 4 and `validate-plan` passes per Step 5.
  The Plan spawn point drops its "state the cards and spawn before writing" instructions.
  It keeps the override of the template's no-review-logic line and the extra fork line.
- **Ready marker removed**: the `ready_marker` directive marker, `twilldirective.Values.ReadyMarker`, `DiscussionReadyMarker`/`PlanReadyMarker` and their `Remove…` functions are gone.
  So are the `twill-write-scope` stencil, `twilldirective.WriteScope` and `WriteScopeMarkerName`, and the `{{.twill_write_scope}}` marker in `loom-template-discussion.md`.
  The write-scope paragraph existed only to add the marker to the Discussion writer's write scope.
  With the marker line now blank, a `single` or `seats` prompt renders the same bytes as before.
- **Seams removed**: `shedrecipe.Env.PrepareDiscussionSpawn` and `PreparePlanSpawn` only ever removed a stale marker.
  They are dropped along with their `loomcli` wiring, the envkit and fixture stubs, and batten's seam-exclusion rows.
  DiscussionWrite's fresh-spawn preparation is back to the parent-review `PrepareRound` alone, nil without that gate.
  PlanWrite's preparation is back to the plan-directory rotator alone.
- **Docs**: the `internal/twilldirective` package doc, the twill paragraphs in `internal/loomengine`'s `discussion.go` and `plan.go` file docs, and the twill rows of `docs/overview.md`.

## To check by hand

- `lyx shuttle await` stays, since `lyx burler await-review` and other waits share it; only the twill forks stop calling it.
- A hub seeded before this change keeps a stale `twill-write-scope.md` in its stencils directory.
  Nothing reads it any more, and no retirement mechanism removes seeded stencils.
- The twill directive still sits where it did in both templates: between Step 4 and Step 5 in Discussion, and between Step 2 and Step 3 in Plan.
  Its spawn point now points forward to the later steps, so confirm that reading order is clear in a rendered prompt.
