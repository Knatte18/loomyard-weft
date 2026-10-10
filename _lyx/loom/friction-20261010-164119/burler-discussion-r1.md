# Fixing the quick-recipe discussion, round 1: loom's recipe comment misstates PR-Gate's budget

A comment in `contracts/recipes/loom-recipe.yaml` says PR-Gate's "episode Stuck count never resets between rounds, so the budget is five rejection rounds per task".
`episodeStuckCount` in `internal/shedengine/run.go` disagrees.
A Done by any row in the same segment ends the episode, and `PR-Rework` shares `PR-Review` with `PR-Gate`, so the count resets after every rework and `max_bounces: 5` never trips.
The discussion writer trusted the comment and claimed a five-rejection bound for darn, which the round-1 review then had to overturn as BLOCKING.

The comment, or the engine if a per-task budget was the intent, needs reconciling outside this task.
Until that happens, any recipe whose rework row shares the gate's segment will repeat the same false bound.
