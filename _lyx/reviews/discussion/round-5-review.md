---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:75
    summary: D4 states the budget test and the config-applied seam budget more than once (lines 75-79, then again in the added entry), so the test is not stated exactly once
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:191
    summary: The added round-4 entry under Decisions only restates D4 and D8 and carries review-round narrative, which belongs in neither Decisions nor the plan writer's input
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:151
    summary: D8's provided-paragraph calls an addition attributable although the same Decision says --by is an unverified label, and D4's Bound paragraph calls fork judgment and visibility a bound
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:246
    summary: No acceptance criterion pins the residual halt to Shed's generic block for an unwired, ok false or errored Bounces read
---

## Verdicts on the focus questions

1. Budget test: the content is correct and agrees with the code.
   `Routing.Bounces` (`internal/shedengine/progress.go`) returns `episodeStuckCount(...)` and `effectiveMaxBounces(row, r.MaxBounces)`, which skips `BudgetExempt` stucks and falls back to the row's `MaxBounces`, then the shed default.
   Shed blocks at `internal/shedengine/run.go:320` on `count >= effectiveMaxBounces(def, s.MaxBounces)` over the history before appending, so D4's `count >= budget` and "the Nth judged round of N escalates where Shed would block" are right.
   `segmentBounces` (`internal/loomcli/wiring.go`) reads `loomrecipe.Routing()`, and wiring leaves `MaxBounces` zero, so once D3 drops the literal from the review rows the seam would report the engine default, not `review_max_bounces`.
   D4 names exactly that requirement, so the premise is sound.
   The test is not stated once, though: see F1.
2. Circling-continue claim: D4 (line 81-83) conditions the no-block claim on `ok` true and names unwired, `ok` false and errored reads; Open risks (line 237) names all three; the acceptance bullet (line 246) carries the condition.
   Not a defect, except that no acceptance criterion covers the residual path itself: F4.
3. D8 and D10: D8's paragraph mostly states what is provided and keeps the Plan-Write licence as an open risk.
   It still says "attributable" against its own unverified-label sentence: F3.
   D10 introduces no new guard change and needs no bound.

### [MEDIUM] Budget test stated several times

**Location:** `_lyx/discussion/decision-record.md:75-79` and `:191-197`.
**Issue:** D4 states `count >= budget` (line 75), restates that the seam's budget is the config-applied value (line 76), restates the routing projection and the agreement test (line 77), and restates that no separate comparison exists (line 78).
The added entry states the test and the projection a third time.
A plan writer reading two copies has to decide whether they differ.
**Fix:** state the test once in D4, merge lines 76-78 into the one statement that the seam and Shed read one config-applied budget pinned by a test, and delete the added entry (F2).

### [MEDIUM] Added round-4 entry duplicates D4 and D8 and carries review narrative

**Location:** `_lyx/discussion/decision-record.md:191-202`.
**Issue:** Each of its three calls already sits in D4 (test, conditional halt) and D8 (provided, not bound), and the Open risks and Acceptance criteria carry the same content.
Its heading breaks the `Added after Discussion (<by>, <UTC date>)` shape D8 defines for entries.
Its text names round-4 findings and "the last budgeted round", which is deliberation history the plan writer does not need, and it pins a perishable round number.
Completeness-before-leanness check: nothing in it is absent from D4, D8, Open risks or Acceptance criteria, so removing it loses no requirement.
**Fix:** delete the entry.
The round history is already in `_lyx/reviews/discussion/`.

### [LOW] "Attributable" and "bounded by judgment" overclaim

**Location:** `_lyx/discussion/decision-record.md:151` and `:89`.
**Issue:** D8 says an addition is "visible and attributable to a human reader", but line 150 says `--by` is an unverified label, not an identity.
D4's Bound says the extension is "bounded by the fork's judgment and visible in the committed decision files", which names judgment and visibility as the bound while the same sentence says no count caps it.
**Fix:** D8: say the addition is visible and carries its claimed label and date.
D4: say no count caps the decisions and the committed decision files record each one.

### [LOW] Residual halt has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:246` (Acceptance criteria).
**Issue:** D4 and Open risks name the residual halt to Shed's generic `bounce-budget-exhausted` block for an unwired, `ok` false or errored read, but no criterion requires a test that this path still blocks rather than escalating.
**Fix:** add one acceptance bullet: with the seam unwired, `ok` false or erroring at a spent budget, a `CONTINUE` returns Shed's generic block and does not escalate.
