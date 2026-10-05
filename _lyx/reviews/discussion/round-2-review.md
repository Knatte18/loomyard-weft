---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:141
    summary: D8 bound relies on plan review and PR review reading the marked additions, but no Decision or Scope item makes either reviewer do so
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:73
    summary: D4 "count has reached review_max_bounces" is ambiguous on whether the current round's Stuck is counted, so the escalate-versus-Shed-block boundary and the cause of a CIRCLING at round N are not determined
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:76
    summary: Only a continue on a budget escalation is BudgetExempt; a continue on a circling-cause escalation is counted and can be blocked by Shed's generic budget block
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:79
    summary: D1 and D4 bound one continue to one round but state no ceiling on repeated continue decisions, so the total extension past the budget is unbounded
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:119
    summary: D7 says accept passes "once per round" while accept ends the segment; the pass count and the unconverged record's reader are left loose
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:24
    summary: Scope and D12 pin a tally of failing smoke tests that goes stale once any is fixed
---

### [MEDIUM] D8's bound depends on a reader behaviour nothing decides

**Location:** `_lyx/discussion/decision-record.md` D8 Bound, lines 139-142.
**Issue:** `decision add` checks no caller identity and `--by` is an unverified label, so any shell addressing the run, the Plan-Write agent included, can append to the answer key.
D8 names the bound as "plan review and PR review read the marked entries as additions to be checked".
No Decision or Scope item changes either reviewer.
D10 touches only the Plan-Write stencil, and the PR-Review segment is Out of scope.
In the repo, `loom-rubric-plan-review.md` reads the decision record, but the PR-review contracts do not mention it or the `Added after Discussion` heading.
The only bound that exists in the decided work is the committed diff and the heading marker.
**Fix:** Either add a Scope item and Decision that make the Plan-Review rubric flag `Added after Discussion` entries for check, and drop PR review from the bound, or restate the bound as the marker and the committed diff only, with the unreviewed-addition risk listed under Open risks.

### [MEDIUM] D4's budget test has an off-by-one ambiguity

**Location:** `_lyx/discussion/decision-record.md` D4, lines 73-75.
**Issue:** "the segment's bounce count has reached `review_max_bounces`" does not say whether the count includes the Stuck of the round being judged.
`internal/shedengine/run.go` blocks when the counted Stucks, including the one being recorded, reach the budget (the `>=` at line 320), and the Bouncer comment says the Nth judged round blocks on CONTINUE.
So the Bouncer must escalate when the count before this round's Stuck is budget minus one.
Read literally, D4 escalates one round late and Shed's block fires first.
The same ambiguity decides the cause for a `CIRCLING` ruling at round N, which the cause rule ("budget whenever the count has reached") is meant to settle.
**Fix:** State the comparison exactly: the cause is `budget` when the count plus the current round's Stuck would reach `review_max_bounces`, using the same count and comparison as Shed.
Add an acceptance criterion for the boundary round: CONTINUE at round N of N escalates, and a CIRCLING there is `budget`.

### [MEDIUM] A continue on a circling-cause escalation can still hit Shed's block

**Location:** `_lyx/discussion/decision-record.md` D4, line 76.
**Issue:** The exemption covers only a recorded `continue` on a budget escalation.
A `continue` on a circling escalation is an ordinary counted Stuck.
Whether it later blocks depends on F2's boundary: if the count after it reaches the budget, Shed blocks the run for a human, which the Goal forbids.
The existing `circlingReason` already warns that "continue will block on the bounce budget".
The focus question asks that a `continue` is never counted into Shed's block, and D4 settles it only for one cause.
**Fix:** State that a circling-cause `continue` counts normally and cannot reach the block, because the cause is `budget` whenever its Stuck would reach the budget (F2).
Add that the `circlingReason` clause about blocking is removed.

### [LOW] No ceiling on repeated continues

**Location:** `_lyx/discussion/decision-record.md` D1 line 40, D4 lines 76-79.
**Issue:** D1 says a round past the budget needs a recorded decision, and D4 bounds each decision to one round.
Nothing bounds the number of decisions.
The rejected "fresh budget" alternative is dismissed as unbounded, yet a fork that keeps recording `continue` produces the same extension, one round at a time, visible only in the decision files.
Open risks covers an unanswered parent but not an over-eager one for `continue`.
**Fix:** Either state the intended bound (for example, parent `continue` is allowed at most a stated number of times per segment, then only the operator may), or record under Open risks that the extension is bounded only by the fork's judgment and the committed decision files.

### [LOW] D7's accept bound is loose

**Location:** `_lyx/discussion/decision-record.md` D7 Bound, lines 119-120.
**Issue:** "once per round" and "passes only that segment" describe one event, since `accept` ends the segment.
The record says the decision file records the pass "as unconverged with its cause", but not what later reads it.
**Fix:** Say `accept` passes the segment once, at the awaiting Bouncer row, and that the committed decision file is the record plan and PR review can check.

### [NIT] Pinned tally of failing smoke tests

**Location:** `_lyx/discussion/decision-record.md` lines 24 and 172.
**Issue:** "the five failing" restates a count that changes as soon as one test is fixed or another breaks.
**Fix:** Write "the failing loomcli smoke tests", pointing at `go test -tags smoke ./internal/loomcli/`.
