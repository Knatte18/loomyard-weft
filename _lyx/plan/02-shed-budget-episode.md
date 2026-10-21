# Card 2 — shed-budget-episode

**Edit:**
- `internal/shedengine#episodeStuckCount`
- `internal/shedengine#gotoEndsEpisode`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#OutputPointer`
- `internal/shedengine#OutcomeGoto`
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#ReasonBounceBudgetExhausted`
- `internal/shedengine#TestStep_ToldRunIDNamesGotoWithRunID`
- `internal/shedengine#TestBounces_CountAndBudgetInheritance`
- `internal/shedengine#OutcomeReroute`
- `internal/shedengine#Goto`
- `internal/shedengine/doc.go`

**Delete:**
- `internal/shedengine#TestEpisodeStuckCount_GotoResets`
- `internal/shedengine#TestStep_BudgetExhaustionNamesGotoAndGotoRestoresBudget`

**Create:**
- `plan:internal/shedengine#TestEpisodeStuckCount_Boundaries` -> `func TestEpisodeStuckCount_Boundaries(t *testing.T)`
- `plan:internal/shedengine#TestStep_BudgetHaltResumeGrantsOneBudget` -> `func TestStep_BudgetHaltResumeGrantsOneBudget(t *testing.T)`
- `plan:internal/shedengine#TestStep_GenerationStartSkipsBudgetComparison` -> `func TestStep_GenerationStartSkipsBudgetComparison(t *testing.T)`
- `plan:internal/shedengine#Shed.budgetWayForward` -> `func (s *Shed) budgetWayForward() string`

**Uses:**
- `internal/shedengine#gotoTestProducers`
- `internal/shedengine#stuckHistory`
- `internal/shedengine#scriptedStuckShed`
- `internal/shedengine#gotoRequest`
- `internal/shedengine#readStatus`
- `internal/shedengine#gotoShed`
- `internal/shedengine#newTestShed`
- `internal/shedengine#funcProducer`
- `internal/shedengine#seedStatus`
- `internal/shedengine#segmentedRouting`

**Intent:** Shed's budget count implements D1, D3's mark and D6's mark.

D1: `episodeStuckCount` no longer ends an episode at an `OutcomeGoto` entry; it skips it like any other non-Done entry, for `Goto` and `Relaunch`'s goto alike.
A reroute still ends its target segment's episode through `gotoEndsEpisode`, which keeps that one caller; its doc comment and `OutcomeGoto`'s stop saying a goto ends an episode, and `Goto`'s doc comment stops saying its `OutcomeGoto` entry ends the target segment's bounce episode.
A goto still resets what the target row needs to run; that code is untouched here.

D3: `HistoryEntry` gains `BudgetCapped bool` (`json:"budget_capped,omitempty"`), set only on the Stuck entry the budget-exhausted arm of `Shed.stepLocked` appends.
`episodeStuckCount` stops at a `BudgetCapped` entry of the row, without counting it, only when the entry right after it in the history (any producer's) is not an `OutcomeGoto` entry; when a goto entry directly follows, the capped entry is counted and the scan goes on over the spent Stucks.
So a resume (which writes no entry) grants exactly one more budget, and a goto onto the halted run grants none, nor does a resume after such a goto.
`Shed` gains the told `BudgetWayForward string` clause, read through the new `Shed.budgetWayForward`, whose fallback is ``fmt.Sprintf("%s resumes the row and grants it one more budget", runVerb("run", s.RunID))``.
The budget-exhausted reason becomes `bounce budget exhausted for <row>; way forward: <budgetWayForward>`, keeping the `ReasonBounceBudgetExhausted` prefix and naming no goto; that constant's doc comment says the way forward is the told resume clause.

D6: `OutputPointer` and `HistoryEntry` gain `GenerationStart bool` (`json:"generation_start,omitempty"` on the entry), read only on a Stuck verdict; `stepLocked` copies it onto the appended entry.
`episodeStuckCount` counts a row's `GenerationStart` entry and stops right after it, so the count reads 1 once the seed is recorded, whatever the previous generation spent.
The budget arm's condition becomes `!output.BudgetExempt && !output.GenerationStart && episodeStuckCount(...) >= ...`, so a generation-start Stuck is recorded as counted but never takes `budget_capped` and never halts, even over a spent episode.
Bound: only a producer that starts a generation sets the mark, which is the Bouncer's successful seed of a new generation (card 12), once per generation and never on a re-bounce or a degraded seed; Shed trusts the mark as it trusts `BudgetExempt`, and a generation starts only when the row's subject changed since it settled, or, with no settle record, since its last judge spawn, Merge-Review's merge-in being its subject.
`Routing.Bounces` counts through `episodeStuckCount` and follows every rule without a change.

`doc.go`'s goto, bounce-budget and history-field paragraphs are rewritten to these rules, including the two-stops-name-goto sentence, which now names only the missing-producer refusal.

Tests.
`TestEpisodeStuckCount_Boundaries` replaces `TestEpisodeStuckCount_GotoResets` as one table over `gotoTestProducers`: a goto into a segment keeps every row's count; a reroute into a segment still ends it; a capped entry last in the history reads 0 (a resume's grant); a capped entry followed by one Stuck of the row reads 1; a capped entry directly followed by a goto counts the capped entry and every Stuck before it back to the episode start; a capped entry, a Stuck of the row and then a goto reads 1 (the grant's spent count); a `GenerationStart` entry after spent Stucks reads 1, and one more Stuck after it reads 2.
`TestStep_BudgetHaltResumeGrantsOneBudget` replaces `TestStep_BudgetExhaustionNamesGotoAndGotoRestoresBudget` over `scriptedStuckShed(t, 1, ...)`: the halt's persisted error starts with the prefix, carries `budget_capped` on its entry, names `lyx shed run` and no `goto`; a told `BudgetWayForward` replaces the fallback in the reason; a resume (Step) bounces once and the next Stuck halts again; on a second shed, a `Goto` onto the halted row then a Step halts again at once.
`TestStep_GenerationStartSkipsBudgetComparison`, over its own scripted `funcProducer` on `newTestShed` and a `seedStatus` history whose episode is spent (`scriptedStuckShed` can only mark `BudgetExempt`): a Stuck carrying `GenerationStart` bounces (running, not blocked), and its entry carries the mark and no `budget_capped`.
`TestBounces_CountAndBudgetInheritance` gains rows reading `Routing.Bounces`, so the Bouncer's `Bounces` seam is pinned beside Shed's block: after a goto entry into the segment the count stays at the spent Stucks; after a `budget_capped` entry with nothing after it the count reads 0; after a capped entry directly followed by a goto it reads the spent count; after a `GenerationStart` entry it reads 1.
`TestStep_ToldRunIDNamesGotoWithRunID` keeps only its missing-producer half, and its doc comment narrows to that refusal; the budget half moves into `TestStep_BudgetHaltResumeGrantsOneBudget` as a row whose told `RunID` `some-slug` makes the reason name `lyx shed run some-slug`.
`OutcomeReroute`'s doc comment stops saying a reroute ends the episode "as a goto entry does": a reroute alone ends it.

**ImpactSummary:** Budget count and budget-halt reason change for every shed recipe; callers of episodeStuckCount (stepLocked, Routing.Bounces) follow without edits, and loom's Bouncer Bounces seam reads through them.

**Commit:** 2: shed: a goto keeps the spent budget, a budget halt's resume grants one more, a generation start resets the count
