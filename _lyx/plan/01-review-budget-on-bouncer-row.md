# Card 1 — review-budget-on-bouncer-row

**Edit:**
- `internal/shedadapters#BurlerProducer`
- `internal/shedadapters#BurlerProducer.Call`
- `internal/shedadapters#BurlerProducer.doneExit`
- `internal/shedadapters#NewBouncer`
- `internal/shedadapters#Bouncer.Call`
- `internal/shedadapters/doc.go`
- `internal/shedengine#Routing.Bounces`
- `internal/shedadapters#TestBurlerProducer_RoundScan`
- `internal/shedadapters#TestBurlerProducer_Call_DoneReturnsStuckNeverDone`
- `internal/shedadapters#TestBurlerProducer_Gate_FailedGateMapsToStuckWithEmptyPointer`
- `internal/shedadapters#TestBouncer_Clear_ApprovedRunDirClearsAndReseeds`

**Delete:**
- `internal/shedadapters#BurlerProducer.roundBudgetExempt`
- `internal/shedadapters#TestBurlerProducer_Call_BudgetExemptAfterBudgetContinue`

**Uses:**
- `internal/shedadapters#Bouncer.settleUnconverged`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#Bouncer.budgetReached`
- `internal/shedadapters#Bouncer.degrade`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#TestBouncer_Escalation_RecordedDecision`

**Intent:** D1 of the decision record: the review segment's bounce budget is enforced on the Bouncer row alone, so a round the reviewer approved and the fixer's gate passed can never block before the Bouncer judges it (#503).

In `doneExit`, the success return (a done round whose gate passed or had none) sets `BudgetExempt: true` unconditionally; the gate-failed return above it stays counted, with `BudgetExempt` left false.
In `Call`, the unjudged-round hand-back (the Stuck returned when the highest complete round has no verdict) also sets `BudgetExempt: true`.
Delete `roundBudgetExempt` together with its `readCirclingDecision` read; `readCirclingDecision` itself stays, since `unearnedCircling` and `settleUnconverged` read it.
No other member of the package changes behaviour: `settleUnconverged` keeps marking a recorded continue decision's Stuck exempt exactly when its cause is budget, `settleUndecided` keeps reaching `budgetReached` on every CONTINUE with no decision file before a new round is spawned, and `degrade` keeps every judge-call failure a counted Stuck.

Confirm, while editing, that every Stuck path matches D1's Bound, and state the result in the doc comments rather than compensating in code: the Bouncer's seed Stuck, its re-bounce Stuck, every `degrade` exit (the entry-time attach probe, the clear failure, `settle`'s vanished or unparseable verdict, the settled accept, every `judgeCall` failure), a CONTINUE with no decision and a circling-cause continue are all counted on the Bouncer row; only the budget-cause continue is exempt; `escalate` returns Awaiting, which is never counted.
On the Burler row only the gate-failed exit is counted.
So a judge that never recovers halts within `review_max_bounces` counted Bouncer Stucks, a budget continue buys exactly one judged round, and a round whose gate fails on every attempt after a budget continue blocks the run once the Burler row's budget of gate-failed rounds in the generation is spent (AC2, AC2a).

Rewrite the doc comments to the new rule:
`BurlerProducer`'s comment drops the paragraphs on the episode that never resets, the two-row cap, the one-ahead offset and the "accepted limitation" of a later generation, and says instead that both hand-backs are `BudgetExempt`, that the segment's budget on judged rounds is the Bouncer row's `budgetReached` check, and that this row's own `MaxBounces` caps the gate-failed rounds of one generation, since its episode ends at the segment's Done or a goto into it as `episodeStuckCount` applies it.
The in-body comment in `Call` above the unjudged hand-back replaces "each hand-back spends one unit of this row's own bounce budget" with the real bound: the Bouncer's next call judges the round through `judgeCall`, whose failures exit through `degrade` and are counted, so a judge that never recovers halts within the Bouncer row's budget.
`NewBouncer`'s budget rule drops the two-row consequence and the documented-not-compensated offset, and says the Bouncer row's count is the segment's whole budget on judged rounds, fresh in every generation.
The Warn comment at the settled-generation clear in `Bouncer.Call` drops "can spend the leftover budget that halts the run, since the round producer's own bounce episode never resets"; the clear is still logged at Warn for its cost in sessions.
`doc.go`'s Burler paragraph (the sentences on the Stuck being `BudgetExempt` only after a budget continue decision) states the new rule: every hand-back for judgment is exempt, the gate-failed exit is counted.
`Routing.Bounces`'s doc comment in `internal/shedengine/progress.go` drops the clause that a Burler's own episode never resets, which `episodeStuckCount` already contradicts, and says instead that a Burler-round row sits off the main line, so the count reported while it is current is its step's main-line row's; the method's code is unchanged.
Go comments use semantic line breaks; the lint runs in the card gate.

Tests, per PATTERN-test-economy as extensions of the tests that already cover each hand-back:
`TestBurlerProducer_Call_DoneReturnsStuckNeverDone` asserts the completed round's pointer carries `BudgetExempt` true (AC1);
`TestBurlerProducer_RoundScan`'s `CompleteButUnjudgedRoundHandsBackWithoutSpawning` subtest asserts the hand-back's pointer carries `BudgetExempt` true (AC2);
`TestBurlerProducer_Gate_FailedGateMapsToStuckWithEmptyPointer` asserts `BudgetExempt` false, the counted exit (AC2a).
`TestBurlerProducer_Call_BudgetExemptAfterBudgetContinue` is deleted: its five rows all answer true now, which the Done test pins once, and the Bouncer side of a budget continue is already pinned by `TestBouncer_Escalation_RecordedDecision`.
`TestBouncer_Clear_ApprovedRunDirClearsAndReseeds`'s leading comment drops the sentence on the leftover budget that halts the run; its assertions are unchanged.
AC3 and AC4 need no new assertion: the clear test already pins that a re-entered segment re-seeds from round 1, and `lyx loom status` reads the Bouncer row's count, which this card leaves as it is.
The package's test count drops by one, so `cmd/lyx/testdata/test-budget.yaml` stays.

**ImpactSummary:** Two Burler returns flip to exempt, one method and one test are deleted, the Bouncer's code is untouched and six doc comments are rewritten, one of them on `Routing.Bounces` in `internal/shedengine` with its code unchanged; no caller outside `internal/shedadapters` changes.
