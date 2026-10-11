---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9 Bound, Open risks, Acceptance criteria
    summary: At a spent budget a single refusal blocks the run at once with no re-judge, and with Shed's generic block where a valid CONTINUE at the same count escalates to the parent; D9 neither states nor decides this
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D7 Bound
    summary: D7 claims the guard has no unchecked path to Done, but a settled circling accept reaches Done over an unconverged round by design and D7 does not exempt it
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D10, Acceptance criteria
    summary: D10 records a judge entry only for a pass whose shuttle call returned to this Call, so a judgment settled from disk after a crashed Call leaves none, against the criterion of one judge entry per judge pass
---

### [MEDIUM] A refusal at a spent budget blocks without a re-judge and skips the parent escalation

**Location:** `_lyx/discussion/decision-record.md` D9 `Bound`, the first Open risks bullet, and the acceptance criterion on repeated refusals.

**Issue:** The focus question is settled this way: all three sites name the same exit, Shed's budget-exhausted block, and the code agrees.
A refusal returns a degraded Stuck through `degrade` (`internal/shedadapters/bouncer.go`), which never passes through `settleUnconverged`/`settleUndecided`, the only callers of `budgetReached` and `escalate`.
Shed then applies `episodeStuckCount(st.History, ...) >= effectiveMaxBounces(...)` to that counted Stuck (`internal/shedengine/run.go`) and blocks with `ReasonBounceBudgetExhausted`.
So consistency is not the problem.
The problem is that the Bound describes only the steady case, "a judge that keeps producing refused judgments reaches the budget".
`Bounces` uses the same `count >= budget` comparison as Shed.
On a round where the budget is already reached, a valid CONTINUE escalates to the parent with a brief (`EscalationBudget`).
A refused judgment of that same round blocks the run on its first refusal, with no re-judge and with Shed's generic `goto` way forward instead of the escalation.
This contradicts two things the artifact relies on: D9's premise that a refused judgment is re-judged, and the support log's rejection of "Halting the run on a refused judgment" because "a single careless judge pass would stop a run that a re-judge can recover".
It also means the `Bounces` seam, whose stated purpose is that budget exhaustion "escalates to the parent instead of reaching Shed's generic budget block", is bypassed on exactly the path this task adds.
The retired-verdict precedent has the same property, so D9's "adds no new loop or state" holds, but the precedent covers a near-extinct case while D7/D8 add a guard that is expected to fire.

**Fix:** Decide the exit at a reached budget and state it in D9's Bound, in Open risks, and in the acceptance criterion.
Recommended: a refusal on a round whose budget is reached archives the judge outputs as D9 does, then escalates with cause budget through the existing escalation path, with the refusal cause in the brief, so the parent gets the same hand-off a CONTINUE gets.
Alternatively, keep Shed's block and say plainly that at a spent budget one refusal halts the run without a re-judge, and amend the rejected alternative's reasoning to match.

### [LOW] D7's "no unchecked path to Done" overstates the guard's reach

**Location:** `_lyx/discussion/decision-record.md` D7, the sentence ending "so the guard has no unchecked path to Done".

**Issue:** A pending circling accept settles to Done over a CONTINUE or CIRCLING round (`settleUnconverged` in `internal/shedadapters/bouncer.go`), and that round may carry an open BLOCKING finding.
This bypass is intentional, since the operator ruled on it, but D7 states the guard's coverage without exempting it.
A plan writer taking the sentence literally could add the BLOCKING check to the accept path and refuse an operator's accept.

**Fix:** Narrow the sentence to CONVERGED verdicts and add one line to D7's Bound: an escalation accept the parent recorded reaches Done without the guard, bounded by the parent's decision.

### [LOW] A judgment settled from disk after a crashed Call records no judge usage entry

**Location:** `_lyx/discussion/decision-record.md` D10 ("An entry is recorded for every pass whose shuttle call returned a result") against the acceptance criterion "one `judge` entry per judge pass".

**Issue:** D8 now settles a CONTINUE or CIRCLING verdict on disk at Call entry when no live judge is found.
That judge pass ran, but its shuttle result never returned to any Call, so by D10's rule it records nothing.
The acceptance criterion promises one entry per judge pass without that exception, so the two disagree for the crash-replay case D8 explicitly handles.

**Fix:** Qualify the criterion to passes whose result reached a Call, or state in D10 that a judgment settled from disk after a crashed Call leaves no usage entry.

## Focus directive verdicts

- Refusal exit (D9 Bound, Open risks, repeated-refusal criterion): all three name Shed's budget-exhausted block, and a degraded Stuck does reach that block rather than the escalation, as the code confirms.
  The sites agree with each other.
  The remaining gap is F1: a refusal at a reached budget blocks without a re-judge, and the escalation is skipped.
- Key check at Call entry (D8, D7): not a defect.
  D8 states that a CONTINUE or CIRCLING on disk at entry with no live judge is key-checked before it settles, so a lossy ledger never becomes round N+1's baseline.
  The bound is stated (keys only, skipped with a warning on a malformed previous ledger), and an acceptance criterion pins the entry-time case.
  A CONVERGED on disk at entry never settles: `settledGeneration` clears and re-seeds it whether or not it was harvested, which matches D7.
  An escalation decision exists only for a round some Call already harvested and key-checked, so the entry check cannot discard an operator decision over a ledger that was unchecked.
