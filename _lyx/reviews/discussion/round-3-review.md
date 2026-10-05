---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:75
    summary: D4 defines "reached" as counted Stucks plus this round's Stuck equal to the budget, which is not Shed's comparison and contradicts the Nth-round-of-N acceptance criterion
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:77
    summary: D4 says a circling continue cannot hit Shed's block, but an unwired or failed Bounces seam leaves that path open and still halts for a human against the Goal
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:146
    summary: D8's Bound and the Goal rely on plan review and PR review reading entries marked Added after Discussion, but no Decision or Scope item makes any reader check them
---

### [MEDIUM] D4's budget comparison is not Shed's and contradicts the acceptance criterion

**Location:** `_lyx/discussion/decision-record.md:75`, `:225`
**Issue:** Shed blocks when `episodeStuckCount(st.History, ...) >= effectiveMaxBounces` (`internal/shedengine/run.go:320`), where the count is read before the current Stuck is appended.
The Bouncer's seed Stuck already counts one (`internal/shedadapters/bouncer.go:120-123`), so Shed blocks at the Nth judged round of N.
D4 says "reached" uses Shed's own count and comparison, then defines it as counted Stucks plus the Stuck this round adds equal to the budget.
Read literally that fires one round before Shed would block (judged round N-1), uses equality rather than `>=`, and contradicts "the Nth judged round of N escalates" and the acceptance criterion "At round N of a budget of N".
The plan writer cannot tell which boundary is meant, and the cause of a CIRCLING at the last budgeted round depends on it.
**Fix:** State the test as Shed's: the budget is reached when the segment's counted Stucks before this round's Stuck, the seed Stuck included, are `>=` `review_max_bounces`; so the Nth judged round of N escalates, and a CIRCLING ruling at that round has cause `budget`.

### [MEDIUM] The failed-seam path still reaches Shed's block

**Location:** `_lyx/discussion/decision-record.md:77`
**Issue:** D4 states that a circling `continue` "cannot hit Shed's block", and the Goal forbids a human halt on anything the parent can settle.
The same Decision leaves Shed's block as the backstop "for a failed seam read", and `Bounces` is an optional seam (`bouncer.go:103-106`), so an unwired or failing seam turns a spent budget into a `bounce-budget-exhausted` block for a human.
The claim is unbounded as written.
**Fix:** Qualify the claim as holding when the `Bounces` seam reads, and state that a review-segment Bouncer is wired with the seam; a failed read falls to Shed's block and is the one residual halt, named in Open risks.

### [MEDIUM] D8's bound depends on readers no Decision provides

**Location:** `_lyx/discussion/decision-record.md:8`, `:146`
**Issue:** The Goal says plan review and PR review read an added entry as part of the answer key, and D8's Bound says an addition is "visible and attributable".
Only the Plan-Review rubric reads the decision record (`contracts/stencils/loom/loom-rubric-plan-review.md:55-61`), and it reads Decisions as a whole, not the `Added after Discussion` marker.
No reader of the PR-Review segment is named and no Decision or Scope item changes any reviewer.
Plan review treats an added entry as licensing scope, so the bound rests on a human reading history, not on a checking reader.
**Fix:** Restate the Goal's PR-review clause and D8's Bound as only what the decided work provides: the entry lands in `## Decisions`, where Plan-Review already reads it as answer key, and the marked heading and committed diff make it attributable to a human reader; no reviewer checks marked entries.
