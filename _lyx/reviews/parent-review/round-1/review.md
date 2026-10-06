---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 10
    summary: Kept-candidate reasons go in the PR body, while the parallel test-suite-prune-core states them as testtiming keep directives that its tool reads
---

### [BLOCKING] Kept-candidate reasons disagree with test-suite-prune-core's convention

**Location:** Decision 10 ("Stated reasons live in the commit message"), and the first Acceptance criterion.

**Issue:** The two prune runs share one done-when point: the tool's rerun reports no candidate left without a stated reason.
`test-suite-prune-core`, running in parallel and owning `cmd/testtiming`, decided (its D2) that a kept candidate carries a `//testtiming:keep <reason>` directive comment directly above its `func Test…` line.
Its tool then lists such a test under a "Kept" list with the reason, and refuses an empty reason.
Its D1 also widens what the tool judges: a test that spawns only git is no longer excluded, so once core lands, a rerun over this task's packages reports candidates this task's report never showed.
With reasons only in a PR body, the first rerun after both land shows every one of this task's kept candidates as unexplained, and the done-when point no longer holds on `main`.

**Fix:** State each kept candidate's reason as a `//testtiming:keep <reason>` comment above its `func Test…` line, in the spelling core's D2 defines.
The tool at this task's base ignores the comment, so nothing else in Decision 10 changes; the PR body may still group the reasons.
Keep Decision 9 and the "tool stays as measure built it" scope line: this task makes no tool change and is judged against the tool at its base.
Add an open risk that core's D1 adds candidates in these packages after both land, which are not this task's to handle.
