# loom-recipe.yaml comment misstates PR-Gate's rejection budget; max_bounces never trips

`pr-gate-budget-comment` · Note · bug, loom, shed

Imported from [issue #526](https://github.com/Knatte18/loomyard/issues/526).

What happened: a comment in `contracts/recipes/loom-recipe.yaml` says PR-Gate's "episode Stuck count never resets between rounds, so the budget is five rejection rounds per task".
`episodeStuckCount` in `internal/shedengine/run.go` disagrees: a Done by any row in the same segment ends the episode, and `PR-Rework` shares `PR-Review` with `PR-Gate`, so the count resets after every rework and `max_bounces: 5` never trips.
An agent trusted the comment and wrote a five-rejection bound into a design, which the review then had to overturn as blocking.

Why it is abnormal: the documented budget and the engine behaviour contradict each other, and any recipe whose rework row shares the gate's segment inherits the false bound.

What to do: fix the comment, or change the engine if a per-task budget was intended.

Provenance: task `quick-recipe`; note burler-discussion-r1.md.
