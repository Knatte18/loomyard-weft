# Publish recovery: fix route, merge-in abort, config notice, guard

A loom run that stops at Publish or Finalize now recovers through one named `lyx` route, with no `goto Plan-Bouncer` re-run and no hand repair of the pair.

## What changes

- **Publish verify failure names the fix route (#462).**
  A plan verify or `publish_verify` that exits non-zero and whose failure record was written stops Stuck with a reason ending in `lyx loom goto --to Webster-Burler`, then `lyx loom resume`.
  A timeout, an unstartable shell, a dirty tree and a non-zero exit whose record write failed keep today's reasons and name no goto, since a fixer given no record judges the diff blind.
  The failure-record writer now reports whether it wrote.
  The refusal-spec gains a `Publish: verify failed` row, and the orch role's escalation list gains the matching item.
- **Publish and Finalize redo their own parked merge-in (#471).**
  Before the clean check, the merge-in step reads the pair's merge state through fabric.
  A parked `merge-in` whose source is the run's parent branch is the row's own leftover: the row stops any live conflict-session strand of its run, aborts the merge through `MergeAbort`, logs the abort and merges afresh.
  A stop that fails leaves the merge untouched and stops Stuck naming the strand's guid, `lyx reed remove <guid>` and `lyx loom resume`.
  A parked fabric merge with another verb or source stops Stuck naming `lyx fabric merge --continue` and `--abort`; foreign git merge state stops Stuck naming git.
  The dirty-tree refusal no longer lists staged merge paths as uncommitted work.
- **`lyx loom resume` and `lyx loom start` let that leftover through.**
  For a run whose current producer is Publish or Finalize over a parent-sourced parked `merge-in`, `start` proceeds for a halted run and for a `running` one with a dead driver; `resume` proceeds when a parked driver is live, and with none reports its existing no-live-driver refusal.
  Every other row, verb, source and foreign state keeps `merge_in_progress`.
  The current producer and the parent branch are read only after the probe shows this parked merge-in, so a poisoned status file at any other state still reaches the spawned child as before.
- **Finalize's config notice compares three points (#450).**
  `fabricengine.ReadConfigChanges` reports a per-worktree config file only when the task's copy differs from both the fork point and the parent's current tip.
  A parent change synced into the task, a parent change the task never synced, and a task edit identical to the parent's copy yield no notice.
  The notice also names the parent tip.
- **A young run dir of a stopped strand is respawn-eligible (#452).**
  Test-only: the attach and burler half probe tables gain `stopped` rows, since the stop verb's `stopped` record already fixes the behavior.
- **`lyx loom decision add` refuses a task session's label (#453).**
  Under a non-empty strand name other than the run's recorded parent name, both `--by parent` and `--by operator` are refused and nothing is appended.
  The way forward names the recorded parent, or the operator when none is recorded.
  A named caller whose parent cannot be resolved is refused with the error named.
  An empty name, or the recorded parent's own name, appends as before.
  The plan stencil now tells the plan agent to ask its parent to record a scope addition.

Closes #462
Closes #471
Closes #450
Closes #452
Closes #453

## Design decisions

- The hub runs the goto; there is no automatic Go-side route from a blocked Publish to the fixer, which would be an engine change to shed's one-`on_stuck`-per-row routing.
- The goto targets `Webster-Burler` rather than `Webster-Bouncer`, which would spend a judge turn on findings the record already states.
- The merge-in is aborted and redone rather than continued, matching the resolver's contract; the abort discards any parent-sourced parked merge-in at Publish or Finalize entry, including one an operator started by hand, and costs one more conflict session.
- The decision guard reads only the exported strand name and the recorded parent name, so it guards a mistaken call, not an agent that forges the variable.
  A future driver route for recording decisions needs its own label or exemption.
- A `StopConflictSession` error with an empty guid (the strand table could not be read) is returned as an infrastructure error with the merge untouched, not as the guid-naming Stuck reason.
- `MidMerge` now returns the parked record's verb, source and SHAs; `mergeresolve`'s conflict role constant is exported and `agentname` recognises a role's numbered forms.
- `internal/testkit/envkit/envkit.go` adds the three new `Landing` seams to its `nilLegal` map so `TestWire_EverySeamFilled` passes.
- Docs updated in the same change: `doc.go` of `landingshed`, `loomcli` and `fabricengine`, `refusal-spec.md`, `docs/overview.md` and `cmd/lyx/testdata/test-budget.yaml`.

## To check by hand

- `stop-seam-empty-guid-becomes-stuck` in `internal/landingshed/premerge.go`: confirm an empty-guid stop error never reaches the guid-naming Stuck reason and that both producers map it to a fault with no `MergeAbort` call.
- The Burler row's reviewer prompt interpolates the same rubric marker `loomcli` wires `PublishFailureNote` into, so the fixer round sees the Publish failure section.
- The tmux tier of `TestSmokeBootstrapLifecycle` passes run whole; its `died_driver_proceeds_to_the_success_envelope_and_logs_why` subtest fails when the round gate re-runs it alone (`run.lock` missing), a gate defect on `main` and unrelated to this change.
- Not run by hand: the end-to-end recovery of a real blocked Publish, from the `goto --to Webster-Burler` route and from a parked merge-in re-step.
- The abort in a task pair created from a task worktree: no orch runs outside the prime until `orch-any-worktree` lands, so the decision refusal's way forward there names a parent no session holds.
