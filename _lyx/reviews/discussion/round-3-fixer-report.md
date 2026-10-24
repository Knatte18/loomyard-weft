# Fixer report — discussion, round 3

## Changes

All in `_lyx/discussion/decision-record.md`.

- F1, fixed as proposed: D13's ready notice takes no walk. It names a task with an empty `ParentBeam` to the prime's orch, and a beam's task to that beam's own orch only when the beam's worktree is present here and its orch is live. Every other ready task is named in no notice. The Bound now says such a task waits until the operator or the beam's orch reads the board. The D13 acceptance criterion gains the stopped-or-never-started case, named to the prime's orch not at all.
- F2, fixed as proposed: D10 step 4 runs as an executor in `internal/fabricengine/destroy.go` behind the check pipeline. It first commits the weft's uncommitted record content under `fabricengine.ScopedPathspec`, as `Remove` does. Uncommitted content outside that pathspec refuses the step, naming the paths and the way forward. The PATTERN-fabric-git Constraint names the chokepoint for step 4. The D10 criterion gains the uncommitted-record-file case. The D10 Bound now says the run's records stay reachable in weft history, where before it said nothing survived.
- F3, fixed as proposed: D11's first Bound names both unreviewed cases, the conflict session's resolution and a textually clean merge over files both sides changed, and says the `finalize` gate bounds both.
- F4, fixed as proposed: the duplicate D5 sentence is deleted, and its Info log of the parent's dirty paths is folded into the remaining rule.

Self-check changes, all in `_lyx/discussion/decision-record.md`:

- D10 step 4 names only the pipeline's containment, ownership and dirtiness checks, and drops "without force": `promote` takes no force flag, so the refusal is unconditional.
- The step-4 refusal's way forward names committing or discarding the paths and then re-running `lyx beam promote <slug>`. The D10 criterion reaches the refusal.
- The D10 Bound says the records leave every checkout but stay in weft history, on the beam's weft branch and on every member's weft branch forked from it.
- The Scope bullet on notices excepts the ready notice and points to D13.

## Self-check

- fix-proposal, fix-way-forward and fix-consistency (merged), MEDIUM: step 4's "without force" names a force `promote` cannot set. Fixed as proposed: "force" dropped and the refusal made unconditional.
- fix-claims, LOW: the pipeline has no force check (`destroy.go:94`). Fixed as proposed: the step names the three checks.
- fix-consistency and fix-way-forward (merged), MEDIUM and LOW: the new step-4 refusal has no acceptance criterion. Fixed as proposed: a clause added to the D10 criterion. The step-4 refusal is left out of D10's up-front refusal list on purpose, since it fires mid-run and step 4 states it.
- fix-way-forward, LOW: the refusal does not name the re-run. Fixed as proposed: it now names `lyx beam promote <slug>`.
- fix-consistency, LOW: the Scope bullet on notices overstates the walk. Fixed as proposed: the bullet excepts the ready notice.
- fix-consistency, LOW: the Bound's "reachable only in the weft branch's history" misleads on members' weft history. Fixed differently: the Bound is reworded, and the Rationale is left as it was, since it speaks of content, which still holds.

## Disputed

None.

## Deferred

None.
