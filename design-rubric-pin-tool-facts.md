# Discussion rubric: a decision states the observed tool behaviour it rests on

`rubric-pin-tool-facts` · Note · enhancement, stencil, loom

## Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 2.
In `decision-record-replace`, three sessions in different rounds (reviewers 91782c57 turn 20 and e44fb294 turn 14, fixer 2c77f780 turn 21) each built a scratch git repository to learn what `git commit --allow-empty -- <path>` commits when another path is staged.
The record stated the behaviour its decision relied on without the observed result, so each round repeated the experiment.

## Idea

A rubric line in `contracts/stencils/loom/loom-rubric-discussion-review.md`: a decision that rests on how git or another tool behaves states the observed result and where it is pinned.
