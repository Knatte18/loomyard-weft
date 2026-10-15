---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D5 and D6, settle whether a pending entry's held time (such as a held `parent-review`) is a gate-kind span or a separate kind, and whether D6's per-kind sums can count the closure evaluations that shuttle's wait loop re-runs inside that held time (`internal/shuttleengine/wait.go`, the `gatePending` recheck) twice, or leave the held time with no drawn line."
  - "In D5's Carriers paragraph, settle whether the carrier set is defined by the rows whose producer runs a gated shuttle run or by the producers that copy a gate count today (`internal/shedadapters/gateattempts.go`), and whether the Burler review row, whose `internal/burlerengine/review.go` passes a `GateSpec` with `ReviewGateEntry` that re-prompts up to `reviewGateAttempts`, is carried or explicitly excluded with a reason that the first Auto-mode assumption matches."
  - "In D7, settle what the routed-to row's open visit draws right after a gate bounce, given that the open visit is not a history entry (`checklistVisit.index` is -1 in `internal/loomcli/checklist.go`), and whether the D7 acceptance criterion covers that case."
  - "In D3 and D4, settle whether every child row named in D4's stage cell has a defined number under D3's rule, including child rows inside a review segment, and whether D4's examples match the shipped `contracts/recipes/batten-recipe.yaml` and `contracts/recipes/loom-recipe.yaml` row order."
---

Each entry names a decision whose text was revised or left open after round 1.
Check each against the rubric's `Do not flag` list and the symmetry rule before filing.
