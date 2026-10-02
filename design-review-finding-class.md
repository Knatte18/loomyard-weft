## Goal

A review segment stops when it has converged, judged from fresh reviews, and escalates when it circles.
Today the Bouncer judge is a second reviewer: `contracts/stencils/bouncer/bouncer-template-judge.md` opens with "You are a review-gate judge: a reviewer of the target artifacts against the rubric", reads the artifacts first, and `internal/shedadapters/bouncer.go` maps its APPROVED straight to Done.
So "every finding fixed" counts as converged, although the fixes are new text no fresh reviewer has seen.
The prompt is wrong by design, not the agent: it does what it is told (operator, 2026-10-02).

## Seen in rework-generations (2026-10-02)

One Discussion-Burler round found four MEDIUM, two LOW and two NIT; the fixer fixed all eight, three of them new design decisions; the judge checked each fix against the decision record, marked all resolved, and the segment ended without a second fresh review.
With a high-temperature reviewer every round is a new sample, and one sample cannot show convergence.

## Shape

- The judge's role is the operator's: a quick sit-rep agent that decides converged, continue or circling, and does not itself read code or review the artifacts.
- Convergence is mechanical where it can be: converged when the latest fresh review has no finding at MEDIUM or above (with finding classes, no gating-class finding); whether findings were fixed does not count.
  Go reads the severities (and classes) from the round's review file; no LLM is needed for that call.
  The seed entry has no fresh review yet, so a segment always runs at least one Burler round.
- The judge is needed only for circling: the same ledger key reopened, or a fix in one area producing a new MEDIUM in the same area; circling escalates to the operator.
- `max_bounces` stays the upper bound.
- Applies to all three review segments (Discussion-, Plan- and Webster-Review).
- The reviewer's own top verdict (`verdict: APPROVED` with MEDIUM findings) mixes "not blocking" with "converged"; the judge must not read it as converged.
- Depends on `rework-generations`, which also edits the judge stencil (self-contained focus entries).

## Finding classes (from manifest/designs/review-finding-classification.md)

- Add a finding-class field (`design`, `scope`, `decision`, `consistency`) to burler's review contract and the rubrics; `internal/burlerengine/verdict.go` has only severity today.
- End the round loop when a round returns zero gating-class findings, keeping the cap as a backstop.
- Emit per-class counts in the round record, so the same class recurring across rounds is visible.
- Add a line to the Review Round Invariant: class decides who decides and when the loop stops, never whether a finding is fixed.