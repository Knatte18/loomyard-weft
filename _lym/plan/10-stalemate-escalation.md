# Card 10 — The Bouncer escalates a stalemate

**Create:**
- `plan:internal/shedadapters#EscalationStalemate` -> `const EscalationStalemate EscalationCause = "stalemate"`
- `plan:internal/shedadapters#Bouncer.recordPostFixTargetID` -> `func (b *Bouncer) recordPostFixTargetID(round int)`
- `plan:internal/shedadapters#Bouncer.roundUnchanged` -> `func (b *Bouncer) roundUnchanged(round int) bool`
- `plan:internal/shedadapters#Bouncer.stalemated` -> `func (b *Bouncer) stalemated(round int, windowOpen bool) bool`
- `plan:internal/shedadapters#TestBouncer_Stalemate` -> `func TestBouncer_Stalemate(t *testing.T)`

**Edit:**
- `internal/shedadapters#EscalationCause`
- `internal/shedadapters#EscalationCause.valid`
- `internal/shedadapters#writeEscalation`
- `internal/shedadapters#readEscalation`
- `internal/shedadapters#readCirclingFile`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.settleUnlessRefused`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#TestEscalation_RoundTrip`
- `internal/shedadapters#TestRecordCirclingDecision_WritesPendingFile`
- `internal/shedadapters#TestBouncer_DecisionEscalation`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`

**Extends:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shedadapters#designWrongKeys`
- `plan:internal/shedadapters#readRoundSnapshot`
- `plan:internal/shedadapters#updateRoundSnapshot`
- `plan:internal/shedadapters#TargetSnapshot`
- `internal/shedadapters#roundFixerReportPath`
- `internal/shedadapters#Bouncer.reviewFindings`
- `plan:internal/burlerengine#ParseFixerReport`
- `plan:internal/burlerengine#CheckFixerReport`
- `internal/shedadapters#newBouncerFixture`
- `internal/shedadapters#layoutCirclingBouncer`
- `internal/shedadapters#sequencedJudge`
- `internal/shedadapters#decisionBouncer`

**Intent:**
D6: a fifth cause, `stalemate`, that a Go check escalates once `StalemateRounds` rounds in a row left the review target unchanged.

`EscalationStalemate` joins `valid`; the doc comments and the four-cause error texts of `writeEscalation`, `readEscalation` and `readCirclingFile` list five.
`RecordCirclingDecision` and the circling verbs then settle a stalemate escalation as they settle the other four, `continue`'s hard-cap refusal included.
The escalation brief stencil's cause list gains `stalemate`: the review target stayed unchanged for the configured rounds in a row.

`BouncerConfig` gains `StalemateRounds int` (0 turns the check off, which is also the zero value a hand-built config gets) and `Snapshot *TargetSnapshot` (nil: no post-fix id is recorded, so every round counts as changed and the check never escalates).

`recordPostFixTargetID`, called first in `settleUnlessRefused`, ahead of `routeDecisions`, so for every judged round and a decision-routed one too:
when round N's snapshot record exists and holds no `post_fix_target_id`, and `Snapshot` is set, it takes the id from a copy of `Snapshot` whose `Scope` and `Paths` come from the record, and writes it through `updateRoundSnapshot`.
Every later settle of round N, a replay included, reads the recorded value, so a replay after a target edit settles as the harvest did.
A failed id read or write is logged at Warn and leaves the round counted as changed.

`roundUnchanged` is true only when round N's record holds a `target_id` and a `post_fix_target_id` that are equal, and round N's fixer-report parses through `ParseFixerReport`, carries no `spawned: false`, and passes `CheckFixerReport` against the round's findings with the record's `decisions_routed` (an absent flag counts as changed).
Every other case counts as changed: no record, no post-fix id, a skipped fixer, an unparseable or incomplete report.

`stalemated(round, windowOpen)` answers, in order:
- true when round N's own escalation record holds cause `stalemate`, whatever `StalemateRounds` now is, so a replay escalates with that cause again; false when it holds another cause, which the existing branches settle;
- false when `StalemateRounds` is 0;
- false when `windowOpen` and `designWrongKeys` of round N's ledger is empty, the arbiter's ruling: the round settles as today, through `softCapRule`'s accept; an unreadable ledger in the window also yields, to `softCapRule`'s budget escalation;
- otherwise whether the unchanged rounds in a row ending at N, counted down from N and stopping at the first changed round or at the latest earlier round whose escalation record holds cause `stalemate`, number at least `StalemateRounds`.
An escalation of another cause restarts nothing.

`settleUndecided` calls it after the soft-cap auto-accept and before the escalated-record re-escalation, and escalates with cause `stalemate` when it answers true, writing the record and notice through `escalate` and returning Awaiting.
A converged verdict never reaches it, so convergence stays the judge's, and the judge still writes its ledger every round.
Bound: a stalemate escalation is Awaiting like `circling`, never counts against the bounce budget, and ends no segment without an `accept`.
Its notice names both verbs through Card 9's clause.

Tests, tier0:
- `TestBouncer_Stalemate`, one table over `newBouncerFixture` with a fake source-scope `TargetSnapshot` whose `SourceTreeID` answers scripted ids, run directories laid out with snapshot records, fixer-reports and judge answers. Rows:
  - at `StalemateRounds` 2 with CONTINUE verdicts, a second unchanged round in a row escalates `stalemate` and its notice names accept and continue; one unchanged round does not;
  - a changed round between two unchanged ones resets the count; `StalemateRounds` 0 never escalates; a nil `Snapshot` at `StalemateRounds` 2 never escalates;
  - a CONVERGED verdict on an unchanged round converges;
  - a round with no record, with no post-fix id, whose report carries `spawned: false`, whose fixer-written report misses a disposition, or whose report does not parse counts as changed, though its two ids are equal;
  - a replay of a settled round after the target id changes reads the recorded id and settles as the harvest did;
  - an unchanged round at the soft cap with auto-accept on is auto-accepted;
  - an unchanged round in the window with auto-accept off and a ledger whose only open design-wrong entry is LOW is accepted by `softCapRule`;
  - an unchanged second round in the window whose ledger is unreadable escalates with cause `budget`, not `stalemate`;
  - a replay of a round whose pending escalation holds `stalemate` escalates `stalemate` again and runs no judge, at `StalemateRounds` 2 and again with `StalemateRounds` set to 0 and a CONTINUE verdict below the window;
  - a round whose own escalation record holds cause `circling` is settled by the existing branches, not as a stalemate;
  - after a recorded `continue` on a stalemate round, the next unchanged round alone does not escalate; a recorded `accept` returns Done.
- `TestBouncer_DecisionEscalation` gains rows:
  - a `decision`-class round that `routeDecisions` escalates records its post-fix target id;
  - D3's held pair: round N's review carries the still-open held finding with `carries` and leaves out the one it resolved, so the round escalates with cause `decision` and its detail names only the carried finding's id.
- `TestEscalation_RoundTrip` gains a `stalemate` row; `TestRecordCirclingDecision_WritesPendingFile` gains a row recording cause `stalemate`.

**ImpactSummary:** One new escalation cause and one Go check inside `settleUndecided`; with `StalemateRounds` 0 or no `Snapshot` every existing settle path is unchanged, and the circling verbs and decision files accept the fifth cause.
