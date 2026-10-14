# Card 5 — reader of a pending escalation's cause

**Edit:**
- `internal/shedadapters#PendingEscalation`
- `internal/shedadapters#TestPendingCirclingDecision`
- `internal/shedadapters/doc.go`

**Create:**
- `plan:internal/shedadapters#PendingEscalationCause` -> `func PendingEscalationCause(runDir string) (cause EscalationCause, briefPath string, pending bool, err error)`
- `plan:internal/shedadapters#pendingEscalation` -> `func pendingEscalation(runDir string) (round int, cause EscalationCause, briefPath string, pending bool, err error)`

**Uses:**
- `internal/shedadapters#readEscalation`
- `internal/shedadapters#ErrEscalationMalformed`

**Intent:**
D5 needs the cause of a pending escalation, which `PendingEscalation` reads through `readEscalation` and discards.
`pendingEscalation` is `PendingEscalation`'s body, keeping the cause `readEscalation` returns; `PendingEscalation` returns its round, brief path, pending flag and error, unchanged for its callers, and `PendingEscalationCause` returns its cause, brief path, pending flag and error.
A malformed record, an unknown cause included, is still an error wrapping `ErrEscalationMalformed`, and a raw read failure a plain error, as `readEscalation` returns them; card 6 turns either into a refusal.
`doc.go`'s line on the two pending readers names `PendingEscalationCause` beside them.

`TestPendingCirclingDecision` gains a `wantCause` field, set on its "escalated with no decision" row to the cause of its `escalationRecord` fixture.
It gains rows for a `hard-cap` and a `budget` escalation record with no decision, each pending with that cause, and one whose cause is `bogus`, which is malformed.
For every row it also calls `PendingEscalationCause` and asserts that its pending flag, brief path and error match `PendingEscalation`'s, and that its cause is `wantCause`, which is empty where nothing is pending.

**ImpactSummary:** one shedadapters reader gains a sibling that returns the escalation cause; PendingEscalation's results and callers are unchanged.

**Commit:** 5: shedadapters: read a pending escalation's cause
