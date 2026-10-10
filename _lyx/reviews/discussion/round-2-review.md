---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D3 Decision (childWait accumulation) and AC9
    summary: D3 leaves open whether a check that sees the activity stamp change keeps or drops its own gap, and keeping it subtracts sleep that preceded the new activity, falsifying D3's bound claim and delaying a quiet notice by up to the whole sleep
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, first bullet
    summary: The BudgetExempt constraint cites PATTERN-transient-stop and PATTERN-shed-verb-set, which govern the transient mark and the shed verb set rather than BudgetExempt, and its never-on-a-verdict clause contradicts settleUnconverged, which already sets BudgetExempt on a verdict path
---

### [MEDIUM] D3 leaves the drop-versus-add order of a stamp-changing check open

**Location:** `_lyx/discussion/decision-record.md` D3 Decision (the `childWait` accumulation sentences) and AC9.

**Issue:** D3 says `childWait` "accumulates the gaps observed since the activity stamp `judgeAgents` returns last changed, and drops the accumulation whenever that stamp changes".
A check that observes a stamp change also observes its own wall-versus-awake gap over the interval since the previous check, and D3 does not say which of the two wins in that check.
The gap's position inside that interval relative to the new activity is unknown: a machine that slept for eight hours and woke, after which an agent wrote one line and then stalled, yields one check whose gap lies entirely before the new `LastActivity`.
If the plan drops the accumulation and then adds that check's gap, the eight hours are subtracted from `now.Sub(newest)` for every later check, so the quiet notice for the stalled agent is delayed by eight hours of awake idleness.
That contradicts D3's own claim that the accumulation "holds only gaps after the newest activity" and "never exceeds the suspended time behind any of the three deltas".
The other order, drop including this check's own gap, keeps that claim: a sleep that fell after the new activity within the same interval is then counted as idle, which is today's behaviour and only raises the notice no earlier than today.
AC9 ("Newest agent activity that moves after a sleep gap drops the accumulated gap") has the same ambiguity, so no criterion pins the order either.

**Fix:** State in D3 that a check whose activity stamp differs from the previous check's drops the accumulation and does not add its own gap, so the accumulation only ever holds gaps from intervals that begin after the newest activity.
Add a row to AC9 for the case where the gap and the stamp change are observed in the same check: the gap is not subtracted afterwards.

### [LOW] The BudgetExempt constraint cites patterns that do not govern it

**Location:** `_lyx/discussion/decision-record.md` `## Constraints`, first bullet.

**Issue:** The bullet reads "`PATTERN-transient-stop` and `PATTERN-shed-verb-set`: `BudgetExempt` stays declared in `internal/shedengine` and set at the producer boundary, in the Burler adapter, never on a verdict."
`pattern/PATTERN-transient-stop.md` governs the transient mark (`MarkTransient`, `shedtransient`), not `OutputPointer.BudgetExempt`, and `PATTERN-shed-verb-set` governs the generic verb bodies in `internal/shedverbs`.
Neither says anything about where `BudgetExempt` is declared or set.
The "never on a verdict" clause also reads as false against the current code: `settleUnconverged` in `internal/shedadapters/bouncer.go` sets `ptr.BudgetExempt` on the Stuck it returns for a recorded continue decision, which D1's own Bound relies on.
A plan writer holding this bullet as a PATTERN-backed rule either looks for an invariant that does not exist or reads D1's Bouncer path as a violation.

**Fix:** Drop the two pattern names from the bullet and state the constraint on its own terms: `BudgetExempt` stays a field of `shedengine.OutputPointer`, and D1 sets it only in the Burler adapter's two hand-back returns, beside the Bouncer's existing budget-continue exemption.

## Focus verdicts

These record the verdict on each focus question; none produced a finding beyond F1.

1. D1 (Bound, A1, R1): not a defect.
   After a budget continue on round N, `settleUnconverged` returns the only exempt Bouncer Stuck, and `episodeStuckCount` skips it, so the Bouncer row's count stays at the budget it escalated on.
   A completed round N+1 hands back exempt under D1, the Bouncer judges it with no decision file, and `budgetReached` escalates again before any further round, so a continue buys one judged round as the Bound says.
   An unjudged round N+1 hands back exempt and reaches `judgeCall`, whose failures exit through `degrade` with a counted Stuck that the engine compares against the already-spent Bouncer budget.
   A gate-failed round N+1 returns the counted Burler Stuck from `doneExit`, archives the round, and cycles back through round N's decided, exempt Bouncer Stuck, so the Burler row's count is the only counted stop on that cycle, which is what the Bound and AC2a state.
   The Burler row's episode ends at a Done by any row of its segment or a goto into it, per `segmentEnders` and `gotoEndsEpisode`, which matches D1's "counts the gate-failed rounds of one generation".
2. D4 and D5 (turn-start sourcing): not a defect.
   D4 reads turn starts through `SessionSignalParser` from the bytes `pollEventsTick` parses and from `PromptOffset` up to where the tick's read begins, ordered by offset, so a turn start and a turn end in one batch are told apart by position, and an attached Wait starting past `PromptOffset` still sees the prompt's turn start.
   It never reads the shadow's fold, which `logSessionState` advances only after `pollEventsTick` in each tick, so the `waitshadow.go` contract that the loop never branches on the shadow holds.
   The claude parser's `consumed` stops only before unpaired trailing stamp lines, so a turn-start payload is complete in the same read that sees a later Stop payload.
3. D4 Bound (provider with no recognised turn-start line): not a defect.
   The Bound states inside the Decision that a provider writing no turn-start lines never arms the guard, and that a provider writing them only for the load turns makes every turn end skipped with a Warn line until the run's deadline.
4. D3 wall delta (monotonic stripping): not a defect.
   D3 states the wall delta comes from readings with the monotonic part stripped, as `UnixNano` values or `Round(0)` times, because `Time.Sub` over two `time.Now` readings returns the monotonic difference, and AC8a pins the stripping at the helper's inputs.
5. D3 subtracted stamps: settled, with F1 as the open part.
   The quiet notice subtracts from `now.Sub(newest)`, the fallback from `now.Sub` of the later of the newest history entry and the episode's since, and the api-error check from `now.Sub` of each erroring run's own `LastActivity`, matching `judgeAgents` in `internal/battenshed/notice.go`.
   The reset rule is the stamp `judgeAgents` returns changing; `newest` is the maximum of the runs' `LastActivity`, so a gap after it is after every run's own stamp, provided the order in F1 is fixed.
