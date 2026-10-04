---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:D2
    summary: The D2 checker flags an empty overview but not an overview with no entries, which still renders a directive carrying nothing to follow.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D6
    summary: The carry runs after the parent-side merge, but the Decision does not say that a Stuck refusal leaves the merge in place and that the re-run must not repeat the merge.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:D13
    summary: D13 says the emptied anchor directory stays in place but not whether any lyx path ever removes it or whether a later pair reuses it.
---

### [LOW] Entry-less overview passes the checker

**Location:** `decision-record.md` D2 and D4, Acceptance criteria.
**Issue:** D4 treats only whitespace-only content as inactive, and D2's checker reports only an empty or whitespace-only overview.
A `PATTERN.md` holding just the heading and the framing sentence passes both, so every directive is active while carrying zero entries.
D4's closing claim, that an emptied loomyard PATTERN does not slip through, holds for whitespace but not for this near-empty file.
**Fix:** Add "an overview with no entry bullet" to D2's findings and to the Acceptance criteria's fixture list.

### [LOW] Re-run after a Stuck carry is unspecified

**Location:** `decision-record.md` D6.
**Issue:** The carry runs after Finalize's parent-side merge, so the Stuck refusal leaves the merge done.
The Decision tells the operator to reconcile and re-run but does not state that the re-run skips the finished merge and only retries the carry.
**Fix:** State in D6 that the refusal leaves the merge in place and the re-run retries only the carry.

### [NIT] Anchor directory lifecycle only half stated

**Location:** `decision-record.md` D13, third bullet.
**Issue:** The bullet fixes the removal-time behaviour but does not say that nothing in lyx removes the anchor directory afterwards.
**Fix:** Add that the anchor is never removed by lyx and is reused when a pair is added under it again.

### Checked and not flagged

- D6's fork point is defined for a normal `Add` (the commit holding `_lyx/fabric/origin.json`, matching `internal/fabricengine/doc.go`) and skipped with a report for the other three pair origins; task delete, parent delete and identical changes are each decided and each has an Acceptance criterion.
- D9 names the content moves and the inbound-link files in the record itself.
- D14's "see Open risks" resolves to an existing item, and D2's skip and failure bounds are stated.
- No Decision restates a rejected alternative.
