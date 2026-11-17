# Card 11 — The arbiter and the circling verbs settle a stalemate

**Edit:**
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop#TestEscalationEligible`
- `internal/arbiterstop/doc.go`
- `internal/loomcli#circlingWayNot`
- `internal/loomcli#circlingVerb`
- `internal/loomcli#loomCLI.circlingCmd`
- `internal/loomcli/doc.go`
- `contracts/orchrole/orch-role.md`
- `contracts/stencils/loom/loom-template-arbiter.md`

**Intent:**
D6: a stalemate is circling that Go detected, and the same judgment ends it.

`Eligibility` admits a pending escalation of cause `stalemate` beside hard-cap, circling and decision; `budget` stays refused.
Its doc comment and arbiterstop's doc.go list the admitted causes with `stalemate`.
loomcli doc.go enumerates them in two sentences, and both gain `stalemate`:
the one listing the stops the verb's closed list admits, which ends "a pending escalation of cause `hard-cap`, `circling` or `decision`",
and the one saying the table "asks the arbiter at an awaiting stop with a pending hard-cap, circling or decision escalation only".

Bound: with `stalemate` admitted, the arbiter may accept a stalemated segment with findings still open, without the run's parent.
The accept passes through `acceptRound`, whose `CarryOver` seam carries the open findings into the decision record's `## Open risks` where the recipe sets `carry_over`;
`budget` stays refused, so a spent budget still goes to the parent;
and the segment stays Awaiting until an `accept` or a `continue` is recorded.

The circling verbs need no code change to record a stalemate decision (Card 10); their texts name the fifth kind:
`circlingWayNot`, `circlingVerb`'s doc comment and the group, `accept` and `continue` help in `circlingCmd` list "a circling, hard-cap, budget, decision or stalemate escalation", and the group help says a stalemate escalation is one whose review target stayed unchanged for the configured rounds in a row.
`continue` after a stalemate runs another round, and the Bouncer's count restarts after that round.

The orch role's escalation section and the arbiter stencil's per-cause rules treat `stalemate` as they treat `circling`: accept when the open findings are judged settled or not worth another round, continue otherwise.

`TestEscalationEligible` (tier0) gains a row: a pending `stalemate` escalation is eligible.

**ImpactSummary:** One more admitted cause in the arbiter's switch and text in the circling verbs, the orch role and the arbiter stencil; no verb's behavior changes.
