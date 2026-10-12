# Driver arbiter: a cold, stronger agent decides when a driver is stuck

`driver-arbiter` · Task · feature, shed, orch, stencil · running · Webster-Bouncer

## Seen (2026-10-11)

Drivers escalate to the orch more often than the operator wants (operator, 2026-10-11).
Much of what reaches the orch is settled by a fixed rule: a review escalation is accepted unless a finding makes the design wrong, and after round 3 a remaining blocker goes to the operator.
Every such wake reads the orch's whole context (see `alignment-agent`).

## Idea (operator, 2026-10-11)

A stuck driver spawns its own cold, stronger agent that decides for it, with its own stencil.

- Spawned on a Stuck or halt whose way forward is unknown, and on a review escalation a perch's auto-accept does not settle.
- Reads the board entry, the discussion, the latest review rounds and the error, then decides or names the next action.
- Runs cold on a stronger model than the driver, such as `opus[high]`.
- Hands on to the orch only when the call needs an operator decision that neither the board entry nor the discussion records, or when it is unsure.
- Parent review and PR-Gate stay with the orch, since they exist to bring in its context.

## Related

`orch-wake-policy` sets what wakes the orch; this note takes over most of the wakes that remain after it (design calls, unsettled escalations, unknown halts).
`orch-split` splits the orch into a run driver and a conversational orch.

## Open

- What the arbiter may do itself (goto, resume, accept a finding, edit the record) and what it may only recommend.
- Its stencil and model key, and a bound on how many times one run may spawn it.
