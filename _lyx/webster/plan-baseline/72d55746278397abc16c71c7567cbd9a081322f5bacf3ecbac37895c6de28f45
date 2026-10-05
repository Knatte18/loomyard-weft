# Card 8 — The `circling` verbs settle an escalation of either cause

**Edit:**
- `internal/shedadapters/circling.go`
- `internal/shedadapters/bouncer.go`
- `internal/loomcli/circling.go`
- `contracts/specs/refusal-spec.md`
- `internal/shedadapters/circling_test.go`
- `internal/loomcli/circling_test.go`
- `internal/loomcli/circling_integration_test.go`

**Uses:**
- `internal/shedadapters#EscalationCause`
- `internal/shedadapters#readEscalation`

**Intent:**
D7: `lyx loom circling accept|continue` accept an escalation of either cause, and the decision file records the cause beside the decision.

`shedadapters/circling.go`:
- `RecordCirclingDecision`'s precondition becomes "the latest round is escalated": an escalation record exists for it (its cause is the record's), or, for a run directory an older binary wrote, its recorded verdict is `CIRCLING` (cause `circling`).
- `ErrNotCircling` is replaced by `ErrNotEscalated` ("the latest round is not escalated"); grep the repository for `ErrNotCircling` and the old message and retarget every hit.
- `circlingDecisionHeader` gains `cause`; `readCirclingDecision` also returns the cause, reading a legacy file without one as `circling` and rejecting any value outside the two causes.
  Its callers in `bouncer.go` take the extra value and change nothing else in this card.
- The file header says the precondition is an escalated round.

`loomcli/circling.go`:
- The not-escalated refusal reads "the latest judged round at <row> is not escalated", with a way forward saying the verbs apply only to a round the Bouncer escalated (a `CIRCLING` judgement or a spent review budget), and a plain `CONTINUE` round below the budget needs no decision.
  The not-awaiting and not-at-a-Bouncer-row ways forward name an escalation instead of a CIRCLING judgement only.
- The success envelope gains `cause`.
- The group's and both verbs' `Short`/`Long` text describe an escalation of either cause; `continue`'s text says a budget escalation grants exactly one more round, and drops "a spent bounce budget still blocks it".
- The file header's bound paragraph names both causes.

`refusal-spec.md`: reword the `circling: latest round not CIRCLING` row into `circling: latest round not escalated`, and update the trigger and way-forward cells of every `circling:` row that names a CIRCLING judgement only, matching the new messages (`PATTERN-refusal-way-forward`).

What this lets through: an `accept` on a budget escalation passes a segment whose verdict was a plain `CONTINUE`.
The bound stays the existing preconditions: the run awaits at that Bouncer row, the round carries an escalation record the Bouncer wrote, one decision per round, and the committed decision file records it unconverged with its cause.

Tests: `circling_test.go` (shedadapters) records over a budget escalation record with a `CONTINUE` verdict and over a legacy `CIRCLING` verdict with no record, refuses an unescalated `CONTINUE` round with `ErrNotEscalated`, and reads the cause back including the legacy default;
`loomcli/circling_test.go` asserts the reworded refusal and the `cause` key;
`circling_integration_test.go` follows any message it asserts.

**ImpactSummary:** Widens what the two verbs accept from CIRCLING rounds to escalated rounds; the refusal text and one exported sentinel change, and every reference moves in this card.

**Commit:** 8: The circling verbs settle an escalation of either cause
