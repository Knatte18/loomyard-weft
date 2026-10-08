---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:49-50,66,148
    summary: D2 does not say how a marker line is recognised, so the claim that a key cannot end the entry early and the refusal rule for stray markers are not decidable
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:64
    summary: D2 bound is a byte comparison of the slices the write itself cuts from the prior file, which cannot fail, so the stated bound holds nothing
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:6,63,135
    summary: The Goal says fresh findings reach the next producer, but a Plan-Review entry reaches no producer before Webster builds the plan, and the Plan-Review rubric still reads the whole record as licensing scope
---

### [MEDIUM] Marker recognition is unspecified

**Location:** `_lyx/discussion/decision-record.md` D2 (lines 49-50, 66) and the acceptance criterion at line 148.
**Issue:** D2 defines the entry as "everything between the two lines, inclusive" and refuses "an opening marker with no closing one", but it never says what makes a line a marker line.
Three things stay open, and the plan writer must pick them:
whether recognition is whole-line equality after trimming or a substring match, whether it is limited to the `## Open risks` section, and whether markers inside a fenced block count.
The record itself shows why this matters: D2's own text spells `<!-- lyx:carry-over <segment> -->` inline in `## Decisions`, so a whole-file substring scan would find an opening marker there with no closing one and refuse every settle of a record that documents this feature.
The criterion "a key holding a marker string cannot end the entry early" holds only under whole-line recognition.
`parseLedger` (`internal/shedadapters/bouncerfiles.go:231`) accepts any key that is non-empty after trimming, so the key really is unconstrained.
The refusal list also covers an unclosed opening marker only: a closing marker with no opening one, two entries for one segment, and an entry whose closing line names another segment are not covered.
The "at most one entry per segment" guarantee relies on none of them occurring.
**Fix:** State in D2 that a marker is a line equal to the marker text after right-trimming, found only inside the `## Open risks` section and outside fenced blocks.
Extend the refusal to every marker set that is not exactly zero or one well-formed, correctly named pair per segment.
Add a criterion for a record that mentions the marker text inline elsewhere and still settles.

### [MEDIUM] The stated bound on the write cannot fail

**Location:** `_lyx/discussion/decision-record.md` D2, the Bound paragraph (line 64).
**Issue:** The bound is "every byte outside that entry is unchanged, which the write checks by comparing the bytes before and after the entry with the prior file".
The write builds its output as prefix, entry, suffix, cut from the prior file at the offsets it found.
Comparing those slices with the same prior file is true by construction, whatever offsets the scan found, so the check catches nothing the scan gets wrong.
The support log (Rejected alternatives) rejects `discussionparser.Validate` for the right reason, that it checks heading presence only.
The record then replaces it with a check that is weaker, not stronger.
What does bound the write is the one-line rendering of judge-written fields (line 65) and the marker recognition of F1; the record presents neither as the bound.
**Fix:** Restate the bound as what the write can actually verify on the result: re-read the new bytes and require that the sequence of H2 headings equals the prior file's, that each segment has at most one well-formed pair, and that the lines outside the replaced span are equal.
Drop the byte comparison as the named check, or keep it only as the last of these.

### [MEDIUM] The Goal and the Open risks bullet claim a guard the consumers do not give

**Location:** `_lyx/discussion/decision-record.md` Goal (line 6), D2 consumers and rubric sentence (lines 60-63), Open risks (line 135).
**Issue:** The Goal says the fresh findings a segment closes on "reach the next producer through the decision record".
That holds for Discussion-Review (Plan-Write, and rework in a later generation) but not for the other two segments.
Plan-Review's next producer is Webster, and no Webster stencil reads the decision record (none of `contracts/stencils` outside the loom plan, rework, prior-plan, rubric, parent-review, escalation and landing-describe files names it).
Plan-Write has already finished when a Plan-Review entry is written, and rework runs only after a PR-Gate rejection.
So a Plan-Review entry reaches Describe first, after Webster has built the possibly wrong plan fix.
The Open risks bullet names Describe as the guard, which is accurate, but it presents it as a guard before the code exists, and the Goal contradicts it.
Separately, the Plan-Review rubric defines the whole record as the answer key and says "every card's scope is licensed by the decision record" (`contracts/stencils/loom/loom-rubric-plan-review.md:62-64`).
D2 declares the rubrics unchanged and tells Plan-Write never to plan an entry as scope, but a Plan-Review reviewer on a later generation reads the same entry as licensing text, and the record does not say the rubric must ignore `## Open risks` carry-over entries.
**Fix:** Reword the Goal to name the actual consumers per segment, and state in the Open risks bullet that a Plan-Review fix is first checked at Describe, after Webster built it, as the accepted bound.
In D2, either keep the Plan-Review rubric unchanged and say a carry-over entry is not part of the answer key, with a one-sentence rubric change if that needs one, or state why a reviewer cannot read it as licence.

## Focus coverage

I departed from no directive.
The four focus entries were each reached and settled as follows.
Carry-over consumers: Scope lists the Plan-Write, rework and Describe stencils; the claimed guard is partial, see F3.
D1 degraded read: the record covers it.
D1 renders a parse-error line for a missing or unparseable earlier ledger, the Bound excludes a MEDIUM `design` finding in a round whose earlier ledgers did not all read, and an acceptance criterion pins the rendering and the round-1 fallback; `ledgerHistories` and `judgeCall` degrade silently today, so the new line is the record's addition and nothing is left open.
Not a defect.
D2 entry shape and bound: the entry shape and the told segment value are fixed; the marker recognition and the bound are not, see F1 and F2.
D2 lifetime and path form: the record states what a later generation, a skipped review and an entry from a generation that never reached `Done` do, and it fixes anchor-relative paths and the ledgers' round numbers.
A stale path in a committed record is a stated cost, not a defect.
