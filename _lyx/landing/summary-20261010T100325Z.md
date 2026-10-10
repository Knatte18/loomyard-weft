# Loom: Publish and Finalize recover through named lyx routes

A loom run that stops at Publish or Finalize now recovers through one named `lyx` route, with no Plan re-run and no hand repair of the pair.

Closes #462
Closes #471
Closes #450
Closes #452
Closes #453

## What changes

- **Publish verify failure names its route (#462).**
  When a plan verify or `publish_verify` ran, exited non-zero and its failure record was written, Publish's Stuck reason ends with the way forward: `lyx loom goto --to Webster-Burler`, then `lyx loom resume`.
  A verify that timed out, could not start, or exited non-zero with no record written keeps today's reason and names no goto, since a fixer given no record would judge the diff blind.
  The record writer now reports whether it wrote.
  `contracts/specs/refusal-spec.md` gains a `Publish: verify failed` row, and the orch role's escalation list names the route.
- **Publish and Finalize redo their own parked merge-in (#471).**
  Before the clean check, the row reads the pair's merge state through fabric.
  A parked `merge-in` whose source is the parent branch is the row's own leftover: the row stops the run's live conflict-session strand, aborts the merge with `MergeAbort`, logs the abort at Info and merges afresh.
  Any other merge state stops Stuck with its way forward (`lyx fabric merge --continue` or `--abort` for another verb or source; concluding or aborting with git for foreign state) instead of a list of staged files.
- **`lyx loom resume` and `lyx loom start` let the row's own leftover through.**
  The `merge_in_progress` refusal stays, except for a run whose current producer is Publish or Finalize over a parent-sourced parked `merge-in`.
  `start` proceeds for a halted run and for a `running` run whose driver died; `resume` proceeds with a live parked driver, and with none reports its existing no-live-driver refusal.
- **Finalize's config notice compares three points (#450).**
  A per-worktree config file is reported only when the task's tip differs from both the fork point and the parent's current tip.
  The notice also names the parent tip.
- **Stopped young run dir (#452).**
  The stop verb already records `stopped`; this change adds only the missing rows to the attach and burler half probe tables, pinning that such a run dir is respawn-eligible and the half reads gone (or done when its output exists).
- **`lyx loom decision add` refuses a task session (#453).**
  A non-empty strand name that differs from the run's recorded parent name is refused for both `--by parent` and `--by operator`.
  The way forward names the recorded parent, or the operator when none is recorded; the operator's unnamed shell always passes.
  The plan stencil now tells the plan agent to ask its parent to record a scope addition.

## Design decisions to know

- The route in the first item is the hub's command, not an automatic Go-side edge: Publish's Stuck also covers merge, push and dirty-tree faults a human owns, and shed routes one `on_stuck` per row.
  `Webster-Burler` is chosen over `Webster-Bouncer` so no judge turn is spent on findings the record already states.
- The abort discards any parked parent-sourced `merge-in` at Publish or Finalize entry, including one an operator started by hand in the pair.
  The row aborts only after the conflict-session stop succeeds; a failed stop leaves the merge untouched and stops Stuck naming the strand guid, `lyx reed remove <guid>` and `lyx loom resume`.
- A stop error with an empty guid (the strand table could not be read) is returned as an infrastructure error and aborts nothing, so no Stuck reason names a strand it cannot name.
- `refuseOverUnfinishedMerge` reads the parent branch only on its own exception path, so a read failure elsewhere cannot replace the `merge_in_progress` refusal.
- The `decision add` check guards a mistaken call, not an agent that forges the exported name.
  When the parent cannot be resolved, a named caller is refused with the error named; the unnamed operator shell is the way past it.
- `fabricengine.MidMerge` now answers the parked record's verb, source and SHAs; `agentname` recognises a role's numbered forms; `mergeresolve` exports its conflict role constant.
- `internal/testkit/envkit/envkit.go` adds the three new `Landing` seams to its `nilLegal` map, which `TestWire_EverySeamFilled` required.

## To check by hand

- The stop-seam error path in `internal/landingshed/premerge.go`: an error with an empty guid must return an error with no `AbortMerge` call, and only a non-empty guid yields the guid-naming Stuck reason.
  This was an open plan-review finding fixed without a later review.
- Until `orch-any-worktree` lands, for a pair created from a task worktree the `decision add` way forward names a parent no session holds; the operator's unnamed shell still passes.
- Finalize's config comparison assumes the records branches share history, as the current fork-point read does.
- Not run by hand: a real Publish blocked on a failing verify taken through `goto Webster-Burler` and `resume`, and a real parked conflict session aborted and redone at Publish.
