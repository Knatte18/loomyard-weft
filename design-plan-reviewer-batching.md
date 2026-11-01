# The round-1 Plan reviewer gathers in one call a turn

`plan-reviewer-batching` · Task · enhancement, burler, stencil · running · Code-Bouncer

## Seen (research, landed 2026-11-01)

`docs/research/review-turn-audit-2026-11-01/report.md`: the round-1 Plan reviewer on Opus is the largest single cost of the review segment, $15.68 over three runs.
89% of its turns make one call, 1.05 calls a turn, and 33 of its 77 turns a session could have been merged into others, 41% of its weight; the estimated saving is $6.43 of $15.68 (n = 3).
It already carried the batching sentence of `bf00a043`, and the sentence did not move it; the Sonnet rounds, which have no forks, batched better.

## Idea

Find what makes round 1 gather serially and trial one change at a time, each measured on the Plan reviewer's calls per turn and removable turns:
- a different instruction, such as naming the gathers to issue together at the start of the review, in its stencil or as a heddle line;
- running round 1 on the model that batched.
Measured with the scripts of `docs/research/review-turn-audit-2026-11-01`.
