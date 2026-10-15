# Route begin-batch plan conflicts to plan review; stop empty config saves

Two unrelated bugs, bundled into one change.

## What changes

### Begin-batch dispatch refusal (webster, loom)

The blocking-findings refusal of `lyx webster begin-batch`'s dispatch re-resolution was labelled `plan_drifted` although the plan fingerprint had already passed, and it halted the loom run for a human although its way forward is a plan edit.

- The refusal now wraps a new webster sentinel, and its envelope carries `{"plan_conflict": "<cause>"}` in place of `{"plan_drifted": true}`.
  The cause is `plan_defect` when `DispatchScope`'s begun set is empty and `tree_moved` when it is not.
  The message keeps its way forward: edit the plan so the named cards match the tree, `lyx webster rebaseline --card NN`, then begin-batch again.
- On that refusal begin-batch writes a markdown conflict record under `.lyx/webster/` (path declared by `internal/websterengine`, reached through the told `Geometry.ScratchDir`), holding the batch, the cause and each blocking finding.
  Every begin-batch removes a present record first, and `websterengine.Run` removes it once it holds run.lock, so a record exists only when the latest begin-batch of this Webster call refused with `plan_conflict`.
  The record is never tracked.
- The Webster row in `contracts/recipes/loom-recipe.yaml` gains `on_reroute: Plan-Burler-Review`.
  On a stuck outcome, the Webster shed adapter (`internal/shedadapters/webster.go`) reroutes when a conflict record exists whose findings differ from the last routed conflict.
  A record equal to the last routed one halts with a reason that names the way forward and does not claim a plan review took place.
  With no record the outcome is plain `Stuck`, as before.
- `Plan-Burler-Review`'s instructions name the conflict record: each blocking finding in it is a finding of the round, fixed by editing only cards whose batch has not begun.
- `loomRerouteCapWayForward` in `internal/loomcli/wiring.go` is now row-neutral, and the Webster master stencil handles `plan_conflict` as it handles `plan_drifted`: stuck, no retry.
- `contracts/specs/refusal-spec.md` replaces the begin-batch plan-drifted row with a plan-conflict row and gains the loom rows for the reroute, the repeated-conflict halt and the reroute-cap halt.

`record-batch` and `recover-batch` keep `plan_drifted`, since their fingerprint refusals are real drift.
`ErrQuarryUnavailable` at begin-batch keeps its path, with no record and no reroute.
Standalone `lyx webster run` gains no reroute.

Closes #625

### Bare `lyx config` without a terminal (configcli, configengine)

Bare `lyx config <module>` and `lyx config menu` run with no interactive terminal opened an editor that exited at once, and lyx saved the empty buffer as the layer.

- Both paths now refuse when stdin is not a terminal, before any choice is read, editor spawned or file scaffolded, with no sync or commit.
  The refusal names `lyx config <module> --print` to read and `--set key=value` to write.
  The terminal test is injected into `configcli`.
- `configengine.EditPath` treats an empty or whitespace-only editor result as an abort: a scaffolded file is removed, a pre-existing file is restored to its previous bytes, and the call returns `ErrAborted`.
  A layer can therefore no longer be emptied through the editor; removing the file stays the way to clear a sparse layer.
  A comment-only result still saves.

Closes #632

## Design decisions to know

- The cause split rests on the begun set, which `DispatchScope` computes before the current batch is recorded.
  A re-begun first batch is therefore `tree_moved` whether or not its fork landed work.
  A `plan_defect` can also arise when an operator commit lands between plan approval and the first batch.
- Repeat detection compares the set of rendered `Finding.Error` strings, order ignored and batch number and cause left out.
  The adapter writes the last-routed file at verdict time, before shed decides whether the reroute is taken.
  A conflict whose reroute shed caps is thus recorded as routed, and resuming that halt without the hand edit halts again as a repeated conflict.
- One conflict gets at most one plan review while the plan edit leaves its findings' card ids and numbers unchanged.
  An edit that renumbers them reads as a new conflict, and shed's reroute cap bounds that case.
- The rerouted review continues the `plan` run directory as round N+1 of the approved episode.
  It inherits that episode's ledger and cluster exclusions, and a round at or past the plan perch's hard cap settles by the soft-cap rule: a CONTINUE auto-accepts back to Webster, a design-wrong one escalates.
- A conflict record left by a halted Plan-Review stays until Webster's next entry.
  If the operator then runs `goto Plan-Write`, the new plan's review treats its findings as findings.
- A failed record write or removal is logged at Warn and changes neither the refusal nor the run outcome.
- `doc.go` of `internal/websterengine`, `internal/shedadapters`, `internal/configcli` and `internal/configengine`, and `docs/overview.md`, are updated.

## To check by hand

- The menu hub-write guard test in `internal/configcli` (`lyx config menu` with a non-terminal stdin): the guard's subtest for the hub-write path was left unspecified and unseen by a fresh review.
  Confirm it asserts no hub-wide write happens.
- Nothing was run against a live loom run: the reroute from a real `begin-batch` refusal into `Plan-Burler-Review` and back to Webster is covered by unit tests only.
- The new "plan conflict rerouted" row in `contracts/specs/refusal-spec.md` uses the class `policy guard`; confirm that class fits.
- `run-pane-timing`, running in parallel, numbers the status pane from the recipe's fixed rows; whichever lands second must render Webster's backward reroute into Plan-Review.
