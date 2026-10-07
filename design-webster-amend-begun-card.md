# An operator decision can amend the card of a begun Webster batch

`webster-amend-begun-card` · Note · bug, webster

An operator decision made while Webster runs cannot reach the plan card of a batch that has begun.
Seen 2026-10-07 in `bugfix-sessions`: the operator decided that the submit settle of card 1 comes from config, while batch 1 was running.
The decision went into the decision record, but `lyx webster rebaseline --card 01` refused ("the edited plan changes the cards of a batch this run already begun"), and its way forward, `lyx webster reset --to start` then `lyx webster run --fresh`, has no path through loom: the Webster producer always runs with `Fresh: false` (`internal/shedadapters/webster.go`).
An edit left on disk while the master recorded the batch blocked the run with `plan_drifted` until the plan was restored.
So the card still says "hard-coded constant" while the code reads config, and the Webster review sees a deviation the operator approved.

Wanted (operator, 2026-10-07): an operator decision taken mid-Webster lands in the plan and the decision record together, without extra review or PR-Gate steps, whether or not the card's batch has begun.
