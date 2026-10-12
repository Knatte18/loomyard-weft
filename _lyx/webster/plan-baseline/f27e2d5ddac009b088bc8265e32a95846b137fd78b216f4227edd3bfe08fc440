# Card 6 — pending-escalation

**Create:**
- `internal/shedadapters#PendingEscalation`

**Edit:**
- `internal/shedadapters#TestPendingCirclingDecision`
- `internal/shedadapters/doc.go`

**Intent:**
Expose the one fact D2's awaiting eligibility and D5's brief input need from a Bouncer run directory: whether its latest round carries a Bouncer-written escalation that no circling decision has answered yet.

`PendingEscalation`, in `circling.go` beside `PendingCirclingDecision`, resolves the latest round as `PendingCirclingDecision` does and reports pending true, with that round and its escalation brief's path (`escalationPath`), only when the round carries an escalation record (`readEscalation` found it) and no circling decision file exists for it.
A run directory with no round, a round with no escalation record, and a round whose decision file exists, settled or not, all report pending false.
A legacy round escalated only by a recorded CIRCLING verdict reports false: it carries no brief, so the arbiter hands it back to the run's parent as before.
A malformed escalation record is returned as its error.

Extend `TestPendingCirclingDecision`'s layouts with an assertion of `PendingEscalation` on each: escalated with no decision (pending, round and brief path), escalated with a pending decision, escalated with a settled decision, not escalated, and no round.
Add a row for a malformed escalation record returning an error wrapping `ErrEscalationMalformed`.
Name the function in `doc.go` beside the circling decision readers.

**ImpactSummary:** one additive reader in `shedadapters`; the existing test gains assertions, nothing else changes.
