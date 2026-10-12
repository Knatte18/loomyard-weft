# Card 17 — orch-arbiter-escalations

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:**
Tell the orch what reaches it now that an arbiter settles a run's stops first (D13).

In `## Escalations from a child`, add that a run's driver hands a stop it cannot settle to an arbiter first, so an escalation from a driver arrives only when the arbiter handed on, and its stop report carries the arbiter's verdict path and question: read the verdict first and act on its question instead of re-deriving the stop.
Add the fixed review-escalation rule for a hand-on of a review loop: accept unless a finding makes the design wrong, and hand a blocker that remains after round 3 to the operator.
In `## Investigating a stop`, add the verdict to what a fork reads first when the stop report names one.
Name the run's driver as "the driver" throughout, so a later move of these sections leaves the arbiter's wording valid.

Add claims to the orch row of `wordingClaims`: the arbiter's verdict path is read first, in `## Escalations from a child`, and `after round 3` stays.
The existing orch claims keep passing.

**ImpactSummary:** prose in one stencil and its claims; no Go code changes.
