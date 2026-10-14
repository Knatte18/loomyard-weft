# Card 10 — LLM driver stencil: fork before arbiter, the verb decides, no relay after a resume

**Edit:**
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils#wordingClaims`

**Uses:**
- `plan:internal/arbiterstop#Eligibility`

**Intent:**
D7, D8 and D9's driver half, in the stencil the LLM driver runs under; the Go driver already behaves this way.

`shed-template-driver.md`:

- `## Error and interrupted stops`: the paragraph saying a transient `kind` that failed twice goes to `## Arbiter` is replaced: such a stop spawns the one fork below under the repair cap, like a `producer` error, and reaches `## Arbiter` only when the fork's action is an escalation (D7).
- `## Arbiter`: the list of stops keeps a `blocked` stop, an `awaiting` stop whose envelope carries a non-empty `parent_notice` (PR-Gate's included), and an `error` or `interrupted` stop after the fork's action is an escalation, a transient one included; the separate line for a transient stop that failed twice is dropped.
  It adds that the arbiter command decides whether the stop is eligible, so the driver never judges eligibility itself and a refused call spawns nothing (D8).
- `## Arbiter`'s envelope handling splits the refusal case: a refusal at an `awaiting` stop is the planned hand-off, so the driver writes the refusal into the stop report with no claim of failure, relays the `parent_notice` verbatim and parks; a refusal at any other stop takes that stop's existing handling under `## Stopping on a non-running state` or `## Error and interrupted stops` (D8).
  The wake re-run rule is unchanged.
- `## Stopping on a non-running state`'s `awaiting` line says the `parent_notice` is relayed only after the arbiter command hands on, refuses, times out or returns a malformed verdict, and never after a `resume` (D9).
- `## Re-launching`'s binary-watch eligible stops become a `blocked` stop and an escalated `producer`, `bootstrap`, `unseeded`, `interrupted` or transient error.

`wordingClaims`' `shed-template-driver.md` entry keeps every claim and adds five:
a `must` of "reaches `## Arbiter` only when the fork's action is an escalation" in `## Error and interrupted stops` (D7: the transient goes to the fork first);
a `must` of "decides whether the stop is eligible" in `## Arbiter` (D8: the verb is the one eligibility filter);
a `must` of "never after a `resume`" (D9: no relay follows a resume);
a `must` of "planned hand-off" in `## Arbiter` (D8: an awaiting refusal relays the notice as a hand-off, not a failure);
and a `mustNot` of "a transient stop that failed twice." (D7: a twice-failed transient goes to the fork first).

**ImpactSummary:** the LLM driver stencil's arbiter section and its transient rule change wording, pinned by five new claims in the existing stencil claims table.

**Commit:** 10: driver stencil: fork before arbiter, the verb decides eligibility, no relay after a resume
