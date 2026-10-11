# Card 7 — bouncer-auto-accept

**Create:**
- `plan:internal/shedadapters#Bouncer.acceptRound` -> `func (b *Bouncer) acceptRound(round int, ptr shedengine.OutputPointer) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#recordAutoAccept` -> `func recordAutoAccept(runDir string, round int) error`
- `plan:internal/shedadapters#TestBouncer_AutoAccept` -> `func TestBouncer_AutoAccept(t *testing.T)`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#NewBouncer`
- `internal/shedadapters#Bouncer.settleUnconverged`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#circlingDecisionHeader`
- `internal/shedadapters#renderCirclingDecision` -> `func renderCirclingDecision(round int, d CirclingDecision, cause EscalationCause, settled, automatic bool) ([]byte, error)`
- `internal/shedadapters#RecordCirclingDecision`
- `internal/shedadapters#settleCirclingAccept`
- `internal/shedadapters/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedadapters#readEscalation`
- `internal/shedadapters#Bouncer.escalate`
- `internal/shedadapters#Bouncer.budgetReached`

**Intent:**
D11: a perch whose `AutoAccept` is set settles a spent budget as an accept instead of escalating to the parent.

`BouncerConfig.AutoAccept` is a told bool, false by default; card 9 fills it from `loom.yaml`.
`settleUndecided` gains one branch, taken only when every one of these holds: the budget is reached (`budgetReached`), `AutoAccept` is set, `spawned` is true (a harvest: the judge wrote this verdict in this Bouncer call, over the round's fixer report), and the round carries no escalation record (`readEscalation`).
Then it records the decision with `recordAutoAccept` and takes the accept path, `acceptRound`: the carry-over of the round's open findings into the decision record's `## Open risks` under the existing accepted class, the settled record, the approve and commit seams, and Done with the ledger pointer.
It writes no escalation record and no parent notice.
`acceptRound` is the pending-accept tail `settleUnconverged` runs today, moved out unchanged so a parent's accept and an automatic one share it.

`recordAutoAccept` creates the round's circling decision file, refusing to overwrite one, as `decision: accept`, `cause: budget`, `settled: false` and `automatic: true`; `circlingDecisionHeader` gains the `automatic` key, which a parent's `lyx loom circling accept` never sets (`RecordCirclingDecision` renders it false) and which is the only place an automatic accept differs from a parent's.
`settleCirclingAccept` keeps the record's `automatic` value when it marks it settled.

Bounds: the branch fires for the budget cause alone, also over a CIRCLING verdict at a spent budget, as the budget escalation does today.
A replay of an earlier round's recorded CONTINUE (`spawned` false) escalates as today: a replay follows a fix gate exhausted in a later round, whose gate-failed edits stay in the target, or a restart between judging and settling.
A CIRCLING verdict below the spent budget still escalates with cause circling; an unreadable decision or escalation record degrades as today; a bounce seam that errors or reports unknown reads as not spent, so a CONTINUE falls to Shed's generic budget block and halts for a human; an escalation already recorded for the round returns the same Awaiting and writes no automatic accept; a recorded decision settles as recorded whatever `AutoAccept` holds; and a Bouncer with `AutoAccept` false behaves exactly as today.

`TestBouncer_AutoAccept` is table-driven over a Bouncer at a spent budget unless the row says otherwise: a harvested CONTINUE settles Done with a settled automatic accept on disk, the carry-over called with the accepted class, approve and commit called, and no escalation record or notice; a harvested CIRCLING at a spent budget does the same; the flag false escalates with cause budget; an escalation record already on disk returns its Awaiting and writes no decision; a replayed CONTINUE escalates; a CIRCLING below the budget escalates with cause circling; an erroring bounce seam returns a plain Stuck; a recorded parent continue returns its budget-exempt Stuck and a recorded parent accept settles Done with `automatic` false.
`NewBouncer`'s budget rule, `BouncerConfig`'s field doc and `internal/shedadapters/doc.go` state the auto-accept settle and its bounds.
The `internal/shedadapters` row of `test-budget.yaml` rises by the new test.

**ImpactSummary:** the Bouncer's undecided-settle branch and the circling decision file's format; with `AutoAccept` false, which every caller passes until card 9, behavior is unchanged.

**Commit:** 7: shedadapters: a Bouncer may accept a spent budget on its own
