---
verdict: BLOCKING
prior:
  - id: F13
    status: resolved
    answer: "D4 now names a way forward for a pair batten did not make. The seed rewrite, goto, start and resume sequence holds against resolveRecipe, shedengine.Goto and seedAndCommitBootstrap over an existing status file, and the D4 criteria reach each reason."
  - id: F14
    status: resolved
    answer: "The branch fact is settled and recorded under Constraints, and the leftover-task Notes bullet is gone. The faults in the text the fix added are F16 and F17."
  - id: F15
    status: resolved
    answer: "D7 now derives every jacquard block and defaults.jacquard from the tapestry values. The reconcile walk runs the follow before the reinterpretations, and FollowTemplate keeps a unit that differs from its base."
findings:
  - id: F16
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:127
    summary: "D5's motif-run clear serves a state the board does not allow: a finished motif run turns its entry into a Sketch note, and the board lets a Sketch become only a raddle task, never a jacquard one"
    fix: "In D5, delete the motif-run passage (lines 124-131). In its place state: `A finished motif run turns its entry into a Sketch (internal/loomcli/wiring.go, finishesAsSketch), which only a raddle run takes (internal/boardengine/sketch.go, checkDiscussedPromotion), so jacquard builds the parts raddle files and other discussed tasks, and batten gains no motif-run clear for it.` In D5's Bound, cut `, and batten's site also reads the slug's done run for the clear above`. Delete the acceptance criterion `D5: after a done motif run of an entry ... starts (tier 1)` and the Constraints Fact about a motif run's task branch (lines 242-244), which only supports the clear. Keep the batten refusal's `lym batten cleanup --apply` clause: a task whose Sketch the board refused stays a draft task over its done motif run (sketchFinish). If the writer instead wants a Sketch to run jacquard directly, that needs a decision changing checkDiscussedPromotion and checkSketchLabel with its own bound and criteria, which the board entry does not ask for."
  - id: F17
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:130
    summary: "D5 says a start over a standing motif pair meets Worktree-Create's leftover-branch refusal; it meets batten's left-pair start guard first, and the cancel that guard names is refused while the done record stands"
    fix: "If F16's fix removes the clear, delete this sentence with it and change nothing else. Otherwise replace lines 130-131 with: `A teardown that finds the pair not level ends Done with the pair standing. The start guard then refuses the start for the left pair before the clear and before Worktree-Create (internal/battencli/startguard.go), and a cancel is refused while the done record stands at Worktree-Teardown (internal/battencli/cancel.go, childReachedParent). So for a clearing start that refusal names, from the hub's prime worktree, lym batten cleanup --apply, then lym batten cancel <slug> --remove, lym board set-status '{\"slug\":\"<slug>\",\"status\":null}' and lym batten run <slug>.` Add an acceptance criterion: `D5: lym batten run of a jacquard entry over a done motif run whose pair stands is refused naming those steps in that order (tier 0, internal/battencli, over the startGuard fakes).` Unverified: how the origin reconcile inside cancel classifies such a standing pair once cleanup removed the record; the writer reads judgeOrigin's inputs before recording the sequence."
---

### [BLOCKING] D5's motif-run clear serves a state the board does not allow

**Location:** `_lym/discussion/decision-record.md:127` (the passage opens at line 124)

**Issue:** D5 and its criterion assume an entry whose motif run is done can be set to `jacquard` and started.
`finishesAsSketch` (`internal/loomcli/wiring.go`) is true for `motif`,
so the finished run calls `MakeSketch`: the entry becomes a note with the `sketch` label and no recipe.
`checkSketchLabel` refuses that label on a task whose recipe is not `raddle`.
`checkDiscussedPromotion` runs on every board write
and refuses making a discussed note with a record file a task under any other recipe, the label stripped or not.
`TestDiscussedPromotion` pins this with `darn`, and I ran it with `TestMakeSketch`: they pass.
So `lym board upsert` or `promote` to `jacquard` is refused for the entry D5 describes.
The tier 1 criterion cannot be met through the real board.
The state remains only where the board refused the Sketch, or where the entry is removed and filed again.
The fixer's new text builds further on this premise.

### [BLOCKING] The way forward for a standing motif pair is not what the code does

**Location:** `_lym/discussion/decision-record.md:130`

**Issue:** For a clearing start, `refuseAtStart` sets `found` false and runs the guard,
whose `PairParts` check refuses a standing task worktree, records worktree or records branch.
`claimStart`, which clears the run, and Worktree-Create are never reached.
That refusal names `lym batten cancel <slug> --remove`.
A done batten record keeps `current_producer` at `Worktree-Teardown`,
for which `childReachedParent` returns true, so the cancel refuses with "the work reached its parent".
The sequence D5 records therefore loops on those two refusals.
`TestChildReachedParent` and the start guard tests pass and pin both halves.
The Worktree-Create refusal D5 names applies only where the pair is gone and a branch deletion failed.

## Rejected

- Focus 1, Record-Seed's Stuck: not a defect.
  Each pair kind has a way forward that clears it, and the D4 criteria reach each reason.
  Where `lym loom start` itself wakes a parked driver, the closing `lym loom resume` is a retry or a no-op.
- Focus 3, the `profiles.yaml` reinterpretation: not a defect.
  After one reconcile jacquard runs read the hub's tapestry values minus `loom.discussion`,
  and a later one keeps each unit that differs from its base.
- Not raised, because the incremental rule admits only BLOCKING findings on changed hunks:
  D5 does not say whether its refusal covers a first `lym loom step`, which shares `seedAndCommitBootstrap`.
- Not raised, same rule: D3 and D4 falsify the block and loom contract statements of `contracts/specs/shed-recipe-spec.md`,
  and the `board` and `board-policy` skills under `plugins/ly/skills/` list recipes without `jacquard`.
  No criterion names either.
- Not raised, same rule: `blockAwareLast` (`internal/shedverbs/human.go`) tells a block row by a non-empty skip target,
  which D3's last block row lacks, so its "entered" line is lost.
