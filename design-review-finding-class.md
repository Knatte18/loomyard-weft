# Review convergence: the judge decides converged, continue or circling from fresh reviews

`review-finding-class` · Planned · feature

Depends on: [`test-suite-change-amplification`](design-test-suite-change-amplification.md)

## Goal

A review segment stops when it has converged, judged from fresh reviews, and escalates when it circles.
Today the Bouncer judge is a second reviewer: `contracts/stencils/bouncer/bouncer-template-judge.md` opens with "You are a review-gate judge: a reviewer of the target artifacts against the rubric", reads the artifacts first, and `internal/shedadapters/bouncer.go` maps its APPROVED straight to Done.
So "every finding fixed" counts as converged, although the fixes are new text no fresh reviewer has seen.
The prompt is wrong by design, not the agent: it does what it is told (operator, 2026-10-02).

## Seen in rework-generations (2026-10-02)

One Discussion-Burler round found four MEDIUM, two LOW and two NIT; the fixer fixed all eight, three of them new design decisions; the judge checked each fix against the decision record, marked all resolved, and the segment ended without a second fresh review.
With a high-temperature reviewer every round is a new sample, and one sample cannot show convergence.

## Shape

- The judge's role is the operator's: a quick sit-rep agent that decides converged, continue or circling, and does not itself read the artifacts or the code.
- The judge decides all three outcomes; a pure mechanical rule was considered and rejected (operator, 2026-10-02): severities are the reviewer's own noisy labels, a nitpicky MEDIUM would force rounds up to `max_bounces`, and a reworded old finding looks new.
- Go hands the judge the facts instead of the artifacts: per round, the finding count and highest severity (and class, once findings carry one), the trend across rounds, and which ledger keys were reopened.
- Only two things stay mechanical: at least one fresh Burler round before convergence is possible (the seed entry has no fresh review, so it never converges straight away), and `max_bounces` as the upper bound.
- Fixed findings never count as convergence on their own; the fixes are new text, and the judge weighs whether the latest fresh review still found MEDIUM-or-worse.
- Circling escalates to the operator.
- The judge gets its own model key: today it shares loom's `review` key (`opus[effort=high]` in `internal/loomengine/template.yaml`) with the reviewers, which is part of why it acts as one; `judge: sonnet[medium]` (operator: Sonnet 5.5 is fast and cheap now, and the job is narrow).
- Applies to all three review segments (Discussion-, Plan- and Webster-Review).
- The reviewer's own top verdict (`verdict: APPROVED` with MEDIUM findings) mixes "not blocking" with "converged"; the judge must not read it as converged.
## Finding classes (from manifest/designs/review-finding-classification.md)

- Add a finding-class field (`design`, `scope`, `decision`, `consistency`) to burler's review contract and the rubrics; `internal/burlerengine/verdict.go` has only severity today.
- End the round loop when a round returns zero gating-class findings, keeping the cap as a backstop.
- Emit per-class counts in the round record, so the same class recurring across rounds is visible.
- Add a line to the Review Round Invariant: class decides who decides and when the loop stops, never whether a finding is fixed.