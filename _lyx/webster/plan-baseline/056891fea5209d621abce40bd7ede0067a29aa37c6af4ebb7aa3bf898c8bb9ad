# Card 2 — hard-cap-cause

**Edit:**
- `internal/shedadapters#EscalationCause`
- `internal/shedadapters#ErrEscalationMalformed`
- `internal/shedadapters#writeEscalation`
- `internal/shedadapters#readEscalation`
- `internal/shedadapters#readCirclingFile`
- `internal/shedadapters#readCirclingDecision`
- `internal/shedadapters#TestEscalation_RoundTrip`
- `internal/shedadapters#TestReadCirclingDecision`
- `internal/shedadapters#TestRecordCirclingDecision_WritesPendingFile`

**Create:**
- `plan:internal/shedadapters#EscalationHardCap` -> `const EscalationHardCap EscalationCause = "hard-cap"`
- `plan:internal/shedadapters#EscalationCause.valid` -> `func (c EscalationCause) valid() bool`

**Intent:** `hard-cap` becomes a third escalation cause beside `circling` and `budget` (D6), legal in the escalation record and the circling decision file alike.

In `escalation.go`, declare `EscalationHardCap` in the cause constant block, whose comment now says three legal spellings, and add `EscalationCause.valid`, true exactly for the three constants.
`writeEscalation` and `readEscalation` validate through `valid`, and their refusal text names all three values.
`ErrEscalationMalformed`'s doc and `readEscalation`'s doc say "one of the three values".
The file's header comment says the `circling` verbs settle an escalation of any of the three causes.
`EscalationCause`'s doc says `hard-cap` is a round at or past its perch's hard cap with a design-wrong finding open, or a recorded continue there.

In `circling.go`, `readCirclingFile` validates the cause through `valid`, its refusal text naming the three values, and `readCirclingDecision`'s doc says "one of their values" instead of "two".
`RecordCirclingDecision` needs no change: it copies the escalation record's cause, so a decision answering a `hard-cap` escalation carries `hard-cap` (D6).

Tests:

- `TestEscalation_RoundTrip` gains a `hard-cap` row writing and reading back `EscalationHardCap` with its brief and notice.
- `TestReadCirclingDecision` gains a row whose file carries `decision: accept` and `cause: hard-cap` and reads back that decision and cause with no error; before this card the same file is refused as an unknown cause.
- `TestRecordCirclingDecision_WritesPendingFile` gains a row "accept over a hard-cap escalation record": round 1 laid out CONTINUE with `writeEscalationRecord(t, dir, EscalationHardCap)`, accept recorded with cause `hard-cap`, and read back by its existing check.

`TestWriteEscalation_RejectsUnknownCause` and `TestEscalation_MalformedRecordIsAnError`'s `mystery` row keep passing unchanged.

**ImpactSummary:** Readers and writers of two record kinds inside shedadapters widen from two causes to three; no caller outside the package switches on a cause, so nothing else changes.

**Commit:** 2: hard-cap is a third escalation cause in the record and the decision file
