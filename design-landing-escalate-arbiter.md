# A landing fixer escalation goes to the arbiter before the orch

`landing-escalate-arbiter` · Note · enhancement, shed, loom

## Seen (2026-10-24)

A landing-stop halt at `Parent-Align` (renamed from `Merge-In` in 5e449651f) or `Finalize`, the merge fixer's `escalate` report included, goes straight to the orch: `internal/arbiterstop` marks it `LandingHalt` and skips the arbiter, and `landingshed.HandsBack` names it a hand-back.
The fixer escalates when a decision in the task collides with work the parent brought in, the one case with a real judgement in it; the old alignment agent (removed in 5e449651f) rejected 4 of 72 such checks, all valid.

## Idea (operator, 2026-10-24)

A fixer's `escalate` at a landing row goes to the arbiter first, as other stops do: the arbiter is a strong agent that can likely settle a decision collision against the decision record and what landed, and only its hand-on reaches the orch.
A spent cap, a failed or unusable fixer turn and an over-cap change stay as they are.

## Landed since (webster-stop-asked-once)

`ArbiterSeam` has an optional `JudgesFirst func(producer string) bool`; a blocked stop at a row it marks goes to the arbiter, skipping the tackler.
Only the Webster row is marked today.
A landing fixer's escalate would be one more row for the seam, not a new mechanism; the landing-halt rule row still precedes it and would need to yield.
