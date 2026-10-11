---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2, D5
    summary: When the merge base sits above the line, the headroom finding demands a cut to the line, but D2's own pass rule passes at the smaller cut back to the base size, so the named byte count is not what the fixer needs to cut
---

### [MEDIUM] The headroom finding's byte count overstates the cut once main sits above the line

**Location:** `_lyx/discussion/decision-record.md`, D2's Decision (the finding names "the bytes to cut to reach the line") and D5's Rationale ("the round gate's headroom finding demands the larger cut, to the line, and the fixer cuts to that").

**Issue:** D2 fails a change only when the head is larger than the base and larger than the line.
So a failing change passes as soon as the head is at most the base or at most the line, and the smallest passing cut is `head - max(base, line)`.
After a Publish merge-in, `loomshed.NewVerifyGate`'s merge-base reader (built like `darnMergeBase` in `internal/loomcli/darnwiring.go`, which reads `handle.MergeBase(parent)`) resolves to the merged-in parent tip.
The base is then main's overview, and that is the case D5 is written for: main above the line, the branch grown on top of it past the hard cap.
There `base > line`, so the gate passes once the branch cuts its own growth (`head - base` bytes).
But the finding names `head - line`, which is larger by `base - line`, and D5 says the fixer cuts that much.
The fixer is told to cut unrelated wording that main brought in, which is the cost D4's Rationale gives as the reason to trim now ("would have to cut over 650 bytes of unrelated wording").
A fixer who cuts only the branch's own growth passes the gate even though the finding said otherwise, so the finding and the gate disagree.
D1's premise holds under this base: the Webster-Bouncer row has no `skip_seam` and its seed call is an unconditional Stuck (`internal/shedadapters/bouncer.go`), so every loom run reaches at least one Webster-Burler round gate.
The defect is only in the count the finding names.

**Fix:** Decide which rule is meant and make D2 and D5 match it.
If a branch above an over-the-line base only has to back out its own growth, the finding names the bytes to reach `max(base, line)`, and D5's Rationale says the fixer cuts the branch's growth back to the base size.
If the intent is to restore headroom whenever main sits above the line, D2's pass rule changes so that growth above the line fails until the head is at most the line, and D2's Bound and the acceptance rows ("shrank while above the line passes", the merge-in row) are restated to match.
Add an acceptance row for the headroom function with the base above the line, stating the byte count the finding names.

## Focus verdicts

- Base for the round gate: D3 names the task branch's merge base with its recorded parent branch, told through a reader that leaves the round form's comment-lint base and impacted-set command alone, and an acceptance row pins that.
  Branch growth is checked in every pair run, since Webster-Burler always runs at least once.
  D1's premise holds under that base, and the arithmetic in its Rationale checks out for a shared base.
- After a Publish verify failure: a branch that never touched `PATTERN.md` has head equal to base and passes, as D2 and the merge-in acceptance row state.
  The byte count does not match the cut the gate needs (F1).
- Attack surface: the headroom lint uses the merge base, not `verifytree.LatestPass`, so a missing or non-ancestor recorded pass does not affect it.
  A merge-base read error is a returned gate error, and a standalone run skips the lint.
  D3's Bound covers both of those and the fork that skips its card gate, and points to the hard cap as the backstop.
  No finding.
- `PATTERN-refusal-way-forward`: the Constraints entry requires the spec row in the same commit, D3's way forward, and a test that reaches the finding and asserts its way forward.
  No finding.
- `/proc` scope: the grep for `"/proc` across `_test.go` files outside `tmuxkit` returns exactly the six files Scope lists.
  D8 covers each one, and the acceptance line names the same property.
  No finding.
