# Scope added after Discussion amends the decision record

`post-discussion-additions` · Note · enhancement, loom, orch

Seen 2026-10-03 in `agent-self-reflect`: the operator added two scope items after Discussion had been approved, and the hub relayed them to the Plan-Write agent.
On the hub's instruction, Plan-Write put them in the plan overview as "operator additions", and plan review then removed them, because the plan must follow the decision record and the record did not carry them.
PR-Gate rejected the PR for the missing items, and the driver had to amend the decision record by hand before PR-Rework.

Idea (operator): a scope addition after Discussion amends the decision record, not the plan.
The agent that receives it (Plan-Write, or whoever holds the run) extends the discussion with the new decisions before planning on them, even if that is a little messy.
Possible shape: a lyx verb that appends an "operator addition" decision to the record and re-runs the mechanical discussion check, so plan review reads the addition as part of the answer key.
