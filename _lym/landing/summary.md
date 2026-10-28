# Landing merges a moved parent in place; fixer escalate goes to arbiter

A landing now settles a moved parent in Finalize instead of rerouting, and on a loom or darn run a landing fixer's `escalate` reaches the orch only as the arbiter's hand-on.

## Why

A loom Finalize that found the parent's changes touching files the task changed rerouted to Parent-Align and re-entered the whole Code-Review perch.
The perch took long enough for the parent to move again, so a run could loop several times at a full perch's cost.
Separately, a fixer `escalate` at Parent-Align or Finalize went straight to the orch, though the arbiter can often settle a decision collision against the decision record and what landed.

## What changes

- `internal/landingshed` Finalize always merges the parent through its resolver, for every recipe.
  A conflict is resolved by the conflict session under the parent lock; a conflict or overlapping merge goes on to the `finalize` gate, whose failure runs the merge fixer in finalize mode.
  The retry after a merge-in-required error merges, gates and lands instead of rerouting.
- Removed: `Deps.ConflictReroutes`, `Deps.LargeMergeLands`, Finalize's abort-on-conflict merge seam, the overlap note (`overlapnote.go`) and darn's overlap-note review input.
  Finalize still writes the merge record's `Overlap`, which the fixer reads.
- A loom or darn fixer `escalate` halts with a new `EscalateStopPrefix`, read only through `EscalateHalt`.
  `landingshed.HandsBack` does not match it, so the envelope carries no `hand_back` mark and `arbiterstop` treats it as an eligible blocked stop.
  Batten's binary re-step skips it like a landing halt.
- `sheddriver.ArbiterSeam.JudgesFirst` takes the stop's reason as well as its producer.
  Loom's wiring (shared by loom, darn and pawl) marks Webster as before plus the landing escalate, which goes rule table, arbiter, then parent with no tackler call.
- `lym shed status` and the loop envelope gain a recipe-neutral `restep_hold` key, read like `hand_back`.
  The LLM driver stencil never re-launches on `binary_changed` and starts no binary watch at a stop marked `restep_hold`.
- A pawl run keeps its escalate a hand-back: `Deps.EscalateJudgedFirst` is false there, so the halt uses `LandingStopPrefix`, and its way forward names no `lym loom decision add`, which a pawl run refuses.
- Docs follow: orch role stencil, loom and darn recipe comments, Code-Review rubric, landing fixer report spec, loom status spec, driver stencil and guide, `docs/overview.md`, package `doc.go` files, and `PATTERN-shed-stop-chain`, whose arbiter-first clause is now per row and reason.

## Design decisions to know

- What lands with no review in loom and darn, as already in pawl: the conflict session's resolution against the parent's newer work, and a textually clean merge of parent changes to files the task also changed.
  The `finalize` gate over the merged tree bounds both, and a spent or over-cap fixer change still reroutes to a review or halts.
- The conflict session runs while the parent lock is held, so sibling landings onto that parent wait and a landing's wall time can lengthen.
- The arbiter can now let a landing proceed past a decision collision with no orch turn.
  Its own rules, its per-run spawn cap (which also stops an escalate-resume loop) and the re-run fixer's gate and caps bound that.
- The driver's build hand-over row precedes the arbiter-first row, so a new build re-steps an escalate stop once before the arbiter sees it; the same holds for any Go driver start with no open arbiter attempt.
- Unchanged: a spent cap, a failed or unusable fixer turn, an over-cap change, an undecodable attempt record and the reroute-cap halt keep `LandingStopPrefix` and their hand-back.
- A run halted with an escalate reason under `LandingStopPrefix` by an older build stays a hand-back; there is no migration.
  A darn run an older build rerouted on a large merge loses its overlap note once upgraded.

## To check by hand

- Not run: a real loom or darn landing against a parent that moved onto the task's files, through the conflict session and the `finalize` gate.
- Not run: a live arbiter settling a landing escalate with `lym loom decision add` and a `resume` whose goto is the halted row.
- `internal/sheddriver` and the driver stencil: the LLM driver can re-launch a halted escalate stop on a new build, since the discussion left the finding `llm-driver-relaunches-halted-escalate` open.
  Check that the stencil's `restep_hold` rule holds, in `contracts/stencils/shed/shed-template-driver.md`.
