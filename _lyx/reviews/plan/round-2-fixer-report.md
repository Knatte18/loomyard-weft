# Plan-Review round 2 — fixer report

## Fixed

### F1 — the walk writes config into a pair `Add` has not finished wiring

Validated against the code: `Topology.Add` (`internal/fabricengine/add.go`) creates the warp worktree, then the weft, portal and launchers, and only then wires the warp junctions (step 10b) and writes the origin record (10c).
`newPaired` (`fabric.go`) checks only that both directories exist, so `OpenCodeWorktree` succeeds in that window, and `WireJunctionsWith` refuses a real directory where the link goes (`junction.go`).
`PairComplete` (`fabric.go`) already answers "did `Add` finish" (sibling worktree, healthy junctions, origin record), but takes a `*lyxcwd.Location`.

Changes:

- Card 5 creates `fabricengine.PairCompleteAt(path)`, the told-geometry form of `PairComplete`: it resolves the worktree root through `lyxcwd.ResolveWorktree` inside fabricengine and returns `PairComplete`'s answer, with `*ErrMissingPath` for a vanished path.
  `PairComplete` joins its `Uses`; its Intent opener, the ImpactSummary count, and the `openparent_integration_test.go` row (complete pair true, junction removed false, deleted pair `*ErrMissingPath`) are updated.
- Card 7's `reconcileWorktree` gains a new step 1: a pair (not `Main`) that `PairCompleteAt` reports incomplete is skipped and logged before anything is opened or written.
  The skip is stamp-neutral (not reported `skipped`): a pair `lyx batten run` creates is covered by card 10's pair call after `Add`, a `lyx fabric add` pair is Decision 4's bounded escape, and a crashed partial pair must not keep every start verb walking.
  The later steps are renumbered, `PairCompleteAt` joins `Uses`, `doc.go`'s list names the incomplete-pair skip, and `ensure_integration_test.go` gains a row: a pair whose warp `_lyx` junction is removed is left unwritten, with no `_lyx` path at the junction's place, no commit on its records branch, and the stamp written.
- The overview's `walk-geometry` Shared Decision and card 5's Card Index line name the completeness check.

The completeness check is called per pair inside `reconcileWorktree`, not inside `CodeWorktrees`, because `PairComplete`'s junction check loads `fabric.yaml`, and the first listing runs before the hub-wide reconcile (the `walk-geometry` rationale).

### F2 — the batten stamp-before-seed assertion cannot observe its ordering

Validated: `arm` resolves, then `armAt` runs `RefuseUnlessPrime`, `refuseSelfAddress`, `armSeed` and `wire` (`internal/battencli/arm.go`), so after a successful `arm` both stamp and seed exist.
Card 10's second test row now runs `arm` for `run` from one of the hub's pairs against an absent stamp: `RefuseUnlessPrime` refuses before the auto-seed, and the row asserts the refusal, a written stamp and no seed for the run id, which pins that the reconcile runs ahead of `armAt` and its seed.

## Focus verdicts

The reviewer's two focus verdicts (not a defect) need no fix.
While orienting I checked the first independently: `loomengine/discussion_test.go`'s missing-stencils-dir row still names `parent-directive-none`, since `DiscussionSpec` reads the parent directive before `composePrompt` reads the template and, after card 16, the edit directive.

## Disputed

None.

## Deferred

None.
