# Decision record — refusal-route-config-empty

## Goal

Two unrelated bugs, bundled into one run on the operator's word (issues #625 and #632).

1. A blocking finding from `lyx webster begin-batch`'s dispatch re-resolution is labelled `plan_drifted` although the plan did not change, and it halts the loom run for a human although its own way forward is a plan edit a plan-fixing agent can make.
   Label it by its real cause and route it under loom to the Plan-Review rows instead of halting.
2. Bare `lyx config <module>` run without an interactive terminal opens an editor that exits at once, and lyx saves the empty buffer as the layer and reports `edited and synced`.
   With no terminal the bare form refuses and writes nothing, and an empty editor result is never saved over a layer.

## Scope

In:

- webster: a new refusal class for begin-batch's dispatch-time blocking findings, carrying a cause; a Go-written conflict record naming the findings; the master stencil's handling of the new class; the refusal-spec row.
- loom: the Webster row gains an `on_reroute` edge to `Plan-Burler-Review`; the Webster shed adapter takes it when a new conflict record exists; `Plan-Burler-Review`'s instructions name the conflict record; loom's told reroute-cap way forward becomes row-neutral.
- config: a terminal guard on every editor path of `lyx config` (bare `<module>` and `menu`, every layer); `configengine`'s edit refuses to keep an empty result.
- Docs in the same commit: the `doc.go` of `internal/websterengine`, `internal/shedadapters`, `internal/configcli` and `internal/configengine`, plus the items D5 names.

Out:

- `record-batch` and `recover-batch` keep `plan_drifted`: they refuse when the plan's fingerprint changed since the run recorded it or the batch began, which is real drift, and their way forward (`lyx webster restore-plan` or rebaseline) stays an operator call.
- `ErrQuarryUnavailable` at begin-batch keeps its current path: no conflict record and no reroute.
- No new delete-before-reference check at Plan-Bouncer or Batchifier: the plan gates already run it.
- Standalone `lyx webster run` (outside loom) gains no reroute: its stuck outcome stays a stop for the operator, with the refusal's way forward unchanged.
- Webster's master agent still never edits the plan.

## Decisions

### D1. Begin-batch's dispatch refusal gets its own class, `plan_conflict`, with a cause

Decision: the blocking-findings refusal of begin-batch's dispatch re-resolution stops wrapping `ErrPlanDrifted` and wraps a new webster sentinel; its envelope carries `{"plan_conflict": "<cause>"}` in place of `{"plan_drifted": true}`.
The cause is `plan_defect` when `DispatchScope`'s begun set is empty (the tree is still the one the plan gates passed, so the plan was wrong when written or the gate's check disagrees with dispatch's) and `tree_moved` when it is not (the run's own work changed the tree, as in `overview-seats`).
The begun set is `DispatchScope`'s, computed before this begin-batch records its batch, so it holds the current batch on a re-begin but not on a first begin;
a re-begun first batch is therefore `tree_moved` whether or not its fork landed work.
The refusal message keeps its way forward: edit the plan so the named cards match the tree, `lyx webster rebaseline --card NN`, then begin-batch again.
The webster refusal note carries the new class in its fields.
Rationale: at begin-batch the fingerprint check has already passed, so the plan is the one the run recorded; "drifted" names the wrong side.
The begun-set test splits the two board cases with an answer begin-batch already computes.

### D2. Begin-batch writes an ephemeral conflict record

Decision: on a `plan_conflict` refusal, begin-batch writes a markdown conflict record under Webster's scratch dir `.lyx/webster/` (path declared by `internal/websterengine`, reached through the told `Geometry.ScratchDir`), holding the batch number, the cause, and every blocking finding as `Finding.Error` renders it, one per line.
It is written under the mutate lock, after the existing plan-fingerprint re-baseline persist.
A write failure is logged at Warn and changes neither the refusal message nor its class; the run then halts as today, since the adapter finds no record.
`websterengine.Run` removes the record once it holds run.lock and before the auto-rebaseline, so a Run refused with `ErrRunBusy` never deletes the record of the live Run.
Every begin-batch call removes a present record as the verb's first step, before batch-number parsing, plan parsing, lock acquisition, state load and batch derivation, and writes it again only on a `plan_conflict` refusal;
a record therefore exists only when the latest begin-batch of this Webster call refused with `plan_conflict`.
A removal failure, at Run entry or at begin-batch, is logged at Warn and changes neither the refusal nor the Run outcome.
Rationale: routing must rest on a Go-written fact, never on the master's quoted `stuck_reason`.
Removing it once Run holds run.lock and before the auto-rebaseline bounds it to one Webster call, so an auto-rebaseline refusal never sees a stale record;
removing it as every begin-batch's first step keeps a later begin-batch refusal of any other kind, from a bad batch number or a plan parse failure through quarry-unavailable or a fingerprint mismatch, from leaving an earlier record in place.
It is never tracked, so no sync has to carry its write or removal; the plan reviewer runs in the same worktree and reads it as plain text.

### D3. Loom reroutes a Webster stuck outcome with a new conflict record to `Plan-Burler-Review`

Decision: the Webster row declares `on_reroute: Plan-Burler-Review`.
Every blocking dispatch finding class writes the record and routes, not only `delete-before-reference`, since all share the one way forward.
The Webster shed adapter, only on its `websterOutcomeStuck` arm and after its existing cancellation check, asks `websterengine` whether a conflict record exists and whether its findings differ from the last routed conflict.
The comparison is over the set of `Finding.Error` strings as D2 writes them: order is ignored, and the batch number and cause are left out, so a reordered but otherwise equal set of findings is equal.
- a record whose findings differ: write the record's findings as the last routed conflict (a second scratch file declared by `websterengine`) and return `Stuck` with `OutputPointer.Reroute` set and a reason that quotes the stuck reason and names the plan-conflict way forward below;
- a record whose findings equal the last routed conflict: return plain `Stuck`, which halts, with a reason saying the same conflict recurred after a reroute of it was asked for, without claiming a plan review saw it, and naming the plan-conflict way forward;
- no record: return `Stuck` as today.

The plan-conflict way forward is: edit the named pending cards by hand so they match the tree, then `lyx loom resume`, which re-steps the Webster row, whose auto-rebaseline accepts the edited cards.
The adapter writes the last-routed file at verdict time, before shed decides whether the reroute is taken, so a conflict whose reroute shed caps is recorded as routed though no plan review saw it.
Resuming that cap halt without the hand edit therefore halts at once as a repeated conflict, with the same way forward, rather than spending a granted reroute on it.
A successful begin-batch removes the last-routed file too.
Shed's reroute-cap halt is reachable at the Webster row, and its reason ends with loom's told `Shed.RerouteCapWayForward`, one clause for every row.
`internal/loomcli`'s `loomRerouteCapWayForward` becomes row-neutral: run `lyx loom resume` in the task worktree once the cause the halt's reason quotes is cleared, which is once the parent branch has settled for a parent-branch reroute and once the named pending cards are edited by hand to match the tree for a Webster plan conflict.
Since the cap reason quotes the adapter's reroute reason, a capped Webster reroute also names the plan-conflict way forward in its own words.
The adapter's error arms (including `ErrAutoRebaseline` and `ErrPendingAuditFindings`) never reroute.
`Plan-Burler-Review`'s fasit instructions in `contracts/recipes/loom-recipe.yaml` name the conflict record: when it exists, each blocking finding in it is a finding of this round, fixed by editing only cards whose batch has not begun.
The rerouted review continues the existing `plan` run directory as round N+1 of the approved episode, so it spawns a reviewer when the highest round carries a Bouncer verdict, which an approved episode's does.
The rest of the route is the existing ring and main line: `Plan-Burler-Fix`, whose `plan` gate (`ValidatePlan`) checks the cards not done against the current tree and refuses any edit to a done card with `done-card-edited`; `Plan-Bouncer`, which re-approves; `Batchifier`; then `Webster`, whose loom entry sets `AutoRebaseline` and so accepts the edited pending cards.
A reroute is not a halt, so it writes no loom halt note; the webster refusal note still records the refusal.
Bound: a reroute is taken only on a stuck outcome with a conflict record from this Webster call whose findings differ from the last routed ones, so one conflict gets at most one plan review while the plan edit leaves its findings' card ids and numbers unchanged, and none when shed caps its reroute; shed's reroute cap (three per row since its latest Done, and Webster is Done only when its whole run finishes) bounds distinct conflicts in a row, including a conflict whose findings read as new after an edit renumbered them; an edit to a begun batch's cards is refused by the gate's `done-card-edited` and by the auto-rebaseline's `ErrRebaselineCardSetChanged`, which halts as today.
The reroute ends the Plan-Review episode, which resets Shed's bounce count, but the Bouncer's hard cap reads the round number, which continues as N+1.
A rerouted round at or past the plan perch's hard cap therefore settles by the soft-cap rule: a CONTINUE with no design-wrong entry auto-accepts and goes back to Webster, where begin-batch checks again;
a design-wrong CONTINUE escalates with cause hard-cap and the run waits on the operator.
The remaining exposure of the conflict record: a master that goes stuck for another reason after a `plan_conflict` refusal, within the same Webster call, is still rerouted;
the master stencil (D4) bounds this by making the master stop at that refusal.
Under the Go driver (`internal/sheddriver`), a rerouted conflict is no longer a stop the driver's tackler sees; a halted one still is.
Rationale: the refusal's way forward is a plan edit, and Plan-Review is the segment that already edits a mid-run plan under a mechanical gate; `on_reroute` is the shed edge built for a cross-segment, budget-free, capped jump.
The repeat check stops a dispatch-time false positive, the `overview-seats` case, after one plan review when that review's edit leaves the findings' card ids and numbers unchanged;
an edit that renumbers them makes the same defect read as a new conflict, and shed's reroute cap of three bounds that case.
Comparing the rendered strings keeps the record plain text with no second key to keep in step with the checks' detail wording.

### D4. The Webster master stencil handles `plan_conflict` as it handles `plan_drifted`: stuck, no retry

Decision: the master's plan-drift section covers both classes: on `{"plan_conflict": ...}` it writes `outcome: stuck` quoting the refusal verbatim and stops, without retrying or trying another batch.
The section says the loom routes a `plan_conflict` stuck to plan review, and that a standalone run's way forward is the refusal's own.
Rationale: the routing lives in Go (D3), so the master's role is unchanged.

### D5. The refusal spec carries the new rows; the recipe header names the reroute

Decision: `contracts/specs/refusal-spec.md` replaces the begin-batch "plan drifted" row with a "plan conflict" row (class `plan_conflict`, cause `plan_defect` or `tree_moved`, correctness halt at the verb, same way forward), and its loom section gains the reroute of a Webster stuck with a new conflict record and the halt on a repeated one.
The repeated-conflict halt's row carries the way forward D3 gives its reason: edit the named pending cards by hand so they match the tree, then `lyx loom resume`.
The loom section also gains a row for a Webster reroute-cap halt on a plan conflict, a correctness halt whose way forward is the same hand edit, then `lyx loom resume`, quoting the row-neutral told clause D3 gives `loomRerouteCapWayForward`.
The recipe's header comment on the escalate-to-a-human set notes that Webster's stuck reroutes to Plan-Review when a new conflict record exists.
Rationale: `PATTERN-refusal-way-forward` requires the rows in the same commit.

### D6. Every editor path of `lyx config` refuses without a terminal

Decision: bare `lyx config <module>` and `lyx config menu` refuse when stdin is not a terminal, checked before any choice is read, layer dispatched, editor spawned or file scaffolded, and with no sync or commit.
The refusal names the way forward: `lyx config <module> --print` to read, `lyx config <module> --set key=value` to write.
The terminal test is injected into `configcli` so tests drive both answers.
Bound: it guards a caller with no terminal; a caller with a terminal and an editor that exits at once is covered by D7.
Rationale: an agent's shell has no terminal, and every editor lyx resolves either needs one or blocks on a GUI no agent sees.

### D7. An empty editor result is never saved

Decision: `configengine.EditPath` (so `Edit` on every layer, and the hub-wide staging copy) treats a result that is empty or whitespace-only after the editor returns as an abort: a file the call scaffolded is removed, a pre-existing file is put back to its previous bytes, and the call returns `ErrAborted`.
`configcli` then reports the existing `aborted: ... unchanged` refusal, and no sync or commit runs.
Bound: a layer can no longer be emptied through the editor; removing the file stays the way to clear a sparse layer.
Rationale: the board requires that an empty editor result is never saved over a layer, whether or not a terminal is present; a comment-only file is a deliberate edit and still saves.

## Constraints

- `PATTERN-refusal-way-forward`: the begin-batch row changes in `contracts/specs/refusal-spec.md` in the same commit, and a test reaches the new class and each cause.
- `PATTERN-no-denied-recovery`: the named ways forward (`lyx webster rebaseline`, `lyx config --print`, `lyx config --set`) stay outside the agents' deny set.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`: the conflict record and the last-routed file are never tracked and live under `.lyx/webster/`, declared only by `internal/websterengine`; the recipe's fasit text names the record's path, and a test pins it equal to the `websterengine` declaration.
- `PATTERN-told-geometry`: `websterengine` takes both files' directory from its told `Geometry.ScratchDir`.
- `PATTERN-shed-driver` and `PATTERN-shed-producer-seam`: the reroute is a verdict from the Webster adapter in `internal/shedadapters`, read by shed through the existing `OutputPointer.Reroute`; no shed engine change.
- `PATTERN-friction-capture`: the webster refusal note keeps being written for `plan_conflict`.
- Row names in `loom-recipe.yaml` are durable resume identities; no row is renamed.
- The Webster master stencil is pinned by `contracts/stencils/claims_test.go`; its claims move with the wording.

## Auto-mode assumptions

- Whitespace-only counts as an empty editor result; a comment-only result does not.
- `lyx config menu` is guarded alongside bare `lyx config <module>`, since it reaches the same editor path.

## Open risks

- A `plan_defect` cause can also arise from an operator commit landing between plan approval and the first batch; the label then says plan, though the tree moved.
- The rerouted review is round N+1 of the approved Plan-Review episode, so it inherits that episode's ledger and cluster exclusions, which may down-weight a conflict finding resembling an excluded cluster; the `plan` gate stays the mechanical backstop for a delete-before-reference conflict.
It also inherits the round number, so a rerouted round at or past the plan perch's hard cap settles by the soft-cap rule (D3): a CONTINUE auto-accepts back to Webster with no fix round, or escalates to the operator when a design-wrong entry is open.
- The last-routed file is written before shed decides the reroute (D3), so a conflict whose reroute shed capped is recorded as routed: resuming the cap halt without the hand edit halts as a repeated conflict, and the reroutes the resume grants serve only a later, different conflict.
- A conflict record that survives a failed removal (D2) can route a later stuck outcome of the same Webster call; the exposure ends at the next successful removal, at Run entry or begin-batch.
- A conflict record left by a Plan-Review that halted stays until Webster's next entry; if the operator then runs `goto Plan-Write`, the new plan's review treats its findings as findings and may force edits nobody asked for.
- `run-pane-timing` (running in parallel) numbers the status pane from the recipe's fixed rows; whichever lands second renders Webster's backward reroute into Plan-Review.

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - card6-menu-hub-write-guard-subtest-unspecified (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- begin-batch with a blocking dispatch finding and an empty begun set refuses with `plan_conflict: plan_defect`; with a begun batch, `plan_conflict: tree_moved`; both messages keep the rebaseline way forward, and neither carries `plan_drifted`.
- Both refusals write the conflict record with the batch, cause and each blocking finding; a quarry-unavailable refusal writes none; a failed record write leaves the refusal message and class unchanged.
- A begin-batch that refuses for a reason other than `plan_conflict` (an invalid batch number, a plan parse failure, quarry-unavailable, a fingerprint mismatch) leaves no conflict record, including one an earlier begin-batch of the same Webster call wrote.
- record-batch and recover-batch fingerprint refusals still carry `plan_drifted: true`.
- A successful begin-batch removes a present conflict record and last-routed file; `websterengine.Run` removes a present conflict record once it holds run.lock and before its auto-rebaseline, and a Run refused with `ErrRunBusy` leaves the record in place.
- The Webster shed adapter, on a stuck outcome: returns `Stuck` with `Reroute` set for a record whose findings differ from the last routed ones and records them; returns plain `Stuck` for a record whose findings equal them, a reordered but otherwise equal set counting as equal, with a reason naming the way forward (edit the named pending cards, then `lyx loom resume`); returns plain `Stuck` with no record; and returns the cancellation, not a verdict, for a cancelled run.
- The adapter's reroute and repeated-conflict reasons both name the plan-conflict way forward, and the repeated-conflict reason does not claim a plan review took place.
- A Webster reroute capped by shed halts blocked with a reason naming the plan-conflict way forward (edit the named pending cards by hand, then `lyx loom resume`), and loom's told reroute-cap clause names no parent branch for it; resuming that halt with no plan edit halts as a repeated conflict and does not reroute.
- An auto-rebaseline refusal with a stale conflict record on disk halts and does not reroute.
- The loom recipe validates with Webster's `on_reroute: Plan-Burler-Review`.
- A reroute into `Plan-Burler-Review` after an approved Plan-Review episode spawns a reviewing round, and its instructions name the conflict record at the path `websterengine` declares.
- The Webster master stencil's claims test pins that a `plan_conflict` refusal ends the run as stuck without a retry.
- `contracts/specs/refusal-spec.md` has the plan-conflict row and no begin-batch plan-drifted row.
- Bare `lyx config <module>` and `lyx config menu` with a non-terminal stdin refuse naming `--print` and `--set`, read no menu choice, spawn no editor, and leave the worktree, local and default layer files byte-identical (absent stays absent), with no sync and no board commit.
- An editor that leaves an empty or whitespace-only file aborts: a scaffolded file is removed, a pre-existing file is restored to its previous bytes, a default-layer edit leaves the board file unchanged with no commit, and no sync runs; a comment-only result saves.

## Notes for the plan writer

- Dispatch refusal: `internal/websterengine/beginbatch.go` (`ErrPlanDrifted` at the blocking-findings return, `DispatchScope`), mapped to the envelope in `internal/webstercli/beginbatch.go`.
- Adapter: `internal/shedadapters/webster.go` maps the run's outcome to a verdict; the reroute mechanism is documented in `internal/shedengine/doc.go` ("The reroute edge and the halting Stuck"); round resolution on entry is `BurlerReviewProducer.Call` in `internal/shedadapters/burlerreview.go`; the reroute-cap halt's reason is built in `internal/shedengine/reroute.go`, and loom's told clause is `loomRerouteCapWayForward` in `internal/loomcli/wiring.go`.
- Plan gate: `ValidatePlan` in `internal/loomshed/gates.go`; auto-rebaseline: `internal/websterengine/autorebaseline.go`.
- Editor path: `internal/configengine/edit.go` (`EditPath`, `Edit`), `internal/configcli/configcli.go` (`editOne`), `menu.go`, `hubwide.go`; a terminal test already exists as `inputIsTerminal` in `internal/loomcli/overlay.go`, and `golang.org/x/term` is a dependency.
