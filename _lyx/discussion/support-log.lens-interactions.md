```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked how the record's decisions act on each other, then checked the acceptance criteria and constraints against them.

**Findings**

1. **BLOCKING** (D8 with the claims "a recorded decision for the round is acted on as today" and "a re-call ... returns the same Awaiting")
   - Summary: D8 escalates a round whose harvested verdict is CONVERGED. The Bouncer's Call-entry path treats a CONVERGED verdict on disk as a settled generation, so a re-call or a resume after the parent's `continue` or `accept` takes the settled-entry branch: harvest, clear, or a new generation. It never reaches the recorded-decision branch the record relies on.
   - Evidence: `settledGeneration` in `internal/shedadapters/bouncer.go:833-843` returns true for any CONVERGED verdict, whatever escalation or decision file exists. `Call` routes `n &gt; 0 &amp;&amp; settledGeneration(n)` to `settledEntry`, whose cases are `entryNewGeneration`, `entryHarvest` (which settles CONVERGED to Done) and `entryClear` (`bouncer.go:427-445`).
   - Today the Bouncer escalates only CONTINUE and CIRCLING rounds (shedadapters doc, "a CONTINUE or CIRCLING round escalates"), so no existing path covers an escalated CONVERGED round.
   - The case the parent is told to use breaks: `lyx loom decision add` commits the record, then the parent runs `continue` or `accept`. With the record changed, re-entry starts an amendment generation. With no subject seam, it clears and re-seeds.
   - Fix: D8 must say how an escalated CONVERGED round re-enters. Options: the escalation also counts in `settledGeneration`, or the verdict is rewritten or set aside when the round escalates. Add an acceptance criterion for a resume after `continue` and after `accept` over a CONVERGED-escalated round.

2. **MEDIUM** (D9 with D8)
   - Summary: D9 has the writer leave a parent-only question as a writer-authored `## Open risks` entry. After the parent answers with `lyx loom decision add` and `continue`, no rule closes or removes that entry. The next reviewer can read it as still open and raise `decision` again, so the run escalates every round until the hard cap.
   - Evidence: D9 records the question "under `## Open risks`". D8's brief tells the parent to "record the answer ... then `lyx loom circling continue`". D8 withholds `decision` findings from the fixer. Nothing in D7–D9 tells the reviewer or the fixer that an `Added after Discussion` entry answering the question resolves the Open risks line.
   - Fix: state, in D7's class definition or in D9, that a question an `Added after Discussion` entry or a `Decision for &lt;slug&gt;:` note answers is no `decision` finding. Also state that the fixer drops the matching Open risks line as an ordinary consistency fix.

3. **LOW** (D8 with the acceptance criterion "`accept` settles, carries the finding into `## Open risks`")
   - Summary: D8 fires at any severity, but the carry-over lists only open ledger findings at MEDIUM or worse. A LOW or NIT `decision` finding is accepted without being carried anywhere, which contradicts the criterion.
   - Evidence: shedadapters doc, Carry-over: "The entry lists the round ledger's open findings at MEDIUM or worse". D8: "at any severity". The support log's rejected alternative argues against MEDIUM-only for this same reason, but the accept path leaves the gap open.
   - Fix: either carry `decision` findings at every severity, or narrow the criterion.

**Pairs checked**
- D1 / D9, D10, D6 (`DecisionNoteTitle`'s readers once the alignment agent is gone): held.
- D1 / D4 (packages deleted while reports keep their record shapes): held.
- D2 / D3 (the gate and its wait key removed together; `alignment_timeout_min` was bounded by `parent_review_wait_min`): held.
- D2 / D9 (both edit `loom-template-discussion.md`: one removes the parent-review text, the other adds the parent-only-question rule): held.
- D2 / D8 (with the gate gone, Discussion-Write hands straight to the perch, so D8 is the only parent stop): held.
- D2's upgrade path / D11's restart rule: held, since only D11 renames a row.
- D5 / D8 (`board-fit` runs only in fanned rounds; a `continue` round is solo under the default fan, and the rubric still defines the class): held.
- D6 / D7 (the same parent-question set in the lens text and in the rubric): held.
- D6 / D10 (one shared "plainly outdated" definition): held.
- D7 / D8 (a tight class definition makes any-severity escalation safe): held, apart from finding 3.
- D8 / auto-accept and the soft-cap rule (precedence stated): held.
- D8 / the hard cap: held.
- D8 / the arbiter's eligibility rule: held.
- D8 / the fix-row notice (silent in the Discussion segment, unchanged in the Plan and Code segments): held, and consistent with Scope Out.
- D8 / re-entry over a CONVERGED verdict: broken (finding 1).
- D8 / D9 (Open risks line after the parent's answer): broken (finding 2).
- D8 / carry-over severity: broken (finding 3).
- D10 / the `decision add` refusal on a darn run before its record exists: held, since the answer arrives by message.
- D10 / D8 (darn's Code-Review perch unchanged): held.
- D11 / D3 (both reconcile migrations): held.
- D11 / `lyx loom peers` row positions: held.
- D11 / Scope Out (fabric's merge-in and `MergeInOnly` untouched): held.
- Constraints `PATTERN-refusal-way-forward` and `PATTERN-shed-recipe-registry` against D8: held.

**Tool calls**
1. `grep` of the Bouncer's Call, clear and decision branches in `internal/shedadapters/bouncer.go`: my context held only the doc text, not the order of the branches in Call.
2. `sed` of `bouncer.go` lines 343-352 and 425-445: to read the clear guard and the `settledEntry` switch the doc only summarises.
3. `grep -A` of the `settledGeneration` predicate: to confirm whether an escalated CONVERGED round counts as settled.
