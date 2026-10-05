---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:75
    summary: D4 names three different counts and budgets for one budget-reached test, and the Bounces seam it relies on reads the budget from the embedded recipe without loom.yaml, so after D3 it reports the wrong budget
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:78
    summary: D4 claims a circling continue cannot hit Shed's block whenever the seam reads, but the seam also reports unknown (ok false) and D4 states the residual halt only for a failed read; the acceptance bullets state the claim unconditionally
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:148
    summary: D8 headed Bound says visibility "bounds" additions while conceding no reviewer checks them, and D10 has the Plan-Write agent add scope that Plan-Review then licenses
---

### [BLOCKING] D4 budget-reached test rests on a budget the seam does not report

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 73-78), Acceptance criteria (line 229).
**Issue:**
D4 says the Bouncer "reads the count through its existing `Bounces` seam" and that "reached" uses "Shed's own count and comparison", and in the same Decision defines it as counted `Stuck`s `>=` `review_max_bounces` of "the segment".
Three things disagree with the code:

- The seam is `Routing.Bounces` in `internal/shedengine/progress.go`, which returns `episodeStuckCount(history, row, ...)` and `effectiveMaxBounces(row, r.MaxBounces)`.
  The count is the Bouncer row's own counted `Stuck`s (`episodeStuckCount` skips every other producer), never the "segment's".
- The seam's routing comes from `loomrecipe.Routing()` (`internal/loomcli/wiring.go` `segmentBounces`), which projects the embedded recipe with `Routing.MaxBounces` left 0.
  D3 removes the literal `max_bounces` from the review rows and has "the loom recipe assembly" apply `review_max_bounces`.
  The seam does not go through that assembly, so after D3 its budget falls to `defaultMaxBounces` (10), not `review_max_bounces`.
  The Bouncer would then escalate at 10 while Shed blocks at `review_max_bounces`, and the Nth judged round of N would block instead of escalate.
- `review_max_bounces` is therefore not the value D4's comparison uses; the seam's budget is.

D4 never states which of these is the test, so the boundary the focus directive asks about is correct in intent (`>=` over the pre-append count, seed `Stuck` included, matching `run.go:320`) but not implementable as written.
**Fix:**
State the test once: the budget is reached when the `Bounces` seam returns `count >= budget`, where `count` is the Bouncer row's own counted `Stuck`s before this round's `Stuck` (seed included) and `budget` is Shed's effective budget for that row.
State as a Decision that the seam's budget must be the same value Shed's engine applies, that is, the config-applied `review_max_bounces`, so D3's assembly path and the seam's routing read one source.
Drop "the segment's" and the separate `review_max_bounces` comparison.

### [MEDIUM] D4 residual halt is stated for a failed read only

**Location:** `_lyx/discussion/decision-record.md` D4 (lines 78-79), Open risks (line 220), Acceptance criteria (line 229).
**Issue:**
`Bounces` is optional and returns `ok` false for unknown (no status file, row not in a segment) as well as an error (`internal/shedadapters/bouncer.go`).
D4 says every review-segment Bouncer is wired and the Open risk names only "a failed `Bounces` seam read".
An `ok` false read also leaves the escalation unreached and falls to Shed's `bounce-budget-exhausted` block, and an unwired Bouncer (any other construction path) does too.
The acceptance bullet "a `continue` on a `circling` escalation below that round is counted and never blocked" is unconditional while the Decision's claim holds only when the seam reads.
**Fix:**
In D4, state the claim as conditional: it holds when the seam returns `ok` true, and the residual halt to Shed's generic block is any unwired, `ok` false or errored read.
Name that residual in the Open risk and add the condition to the acceptance bullet.

### [LOW] D8 labels visibility a bound and Plan-Write can license its own additions

**Location:** `_lyx/discussion/decision-record.md` D8 (lines 146-150), D10 (line 165).
**Issue:**
The Bound paragraph says "What bounds that" is the heading marker and the committed diff, then says no reviewer checks marked entries and the marker makes an addition attributable "to a human reader only".
`contracts/stencils/loom/loom-rubric-plan-review.md` licenses cards from "every Decision" in the decision record without distinguishing marked entries, so the only reader of an added entry is a human after the fact.
D10 directs the Plan-Write agent itself to add the entry before planning on it, so the agent whose plan Plan-Review checks also writes the answer key.
The Goal's claim that plan review "reads it as part of the answer key" is accurate, but D8 presents exposure as a bound.
**Fix:**
Reword D8's Bound to say what is provided: entries are marked and committed, nothing checks them, and a Plan-Write addition is licensed by Plan-Review without a second reader; keep the Open risk.
Remove "What bounds that".
