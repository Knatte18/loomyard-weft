```yaml
lens: code-scope
outcome: reported
```

Task: review the diff under the code-scope lens, tracing each change to the decision record's Goal points and its one Added-after-Discussion entry.

**Decisions**
- G1: Goal "Idea" 1 and "Still open" 1. `pr-continue` passes when a satisfied parent answer stands at the current head, whatever state later asks at that head are in.
- G2: Goal "Idea" 2 and "Still open" 2. The parent can record a satisfied answer at the current head without a delivered ask.
- G3: Goal "Idea" 3 and "Still open" 3. A test runs the ask verb from a live non-orch pane.
- A1: Added after Discussion (parent, 2026-11-23). Item 3 is dropped. Items 1 and 2 stand as committed and delivered. No `--head` flag.

**Changes → decision**
- C1: `internal/landingshed/prreviewask.go`. The body of `ReadPRReviewStanding` scans back to the latest answered, uncleared PR-Review ask at the head, and its doc comment is rewritten to match. Traces to G1.
- C2: `internal/landingshed/prreviewask_test.go`. Five new `TestReadPRReviewStanding` rows. Forced by C1, since they are its tests.
- C3: `internal/landingshed/doc.go`. Two lines stating that the parent's answer at a head stands through later unanswered asks. Forced by C1, since the package doc describes the PR-Review flow C1 changes.
- C4: `contracts/stencils/landing/landing-block-pr-review-autonomous.md` line 20. Tells the session that a satisfied answer stands through a later failed or pending ask. Forced by C1: without it, the session's instructions don't cover what C1 now allows.

**Decision → changes**
- G1: built by C1, with C2, C3 and C4 following from it.
- G2: no change in this range. Under A1 that is correct. Commit 3e8a5eebf ("Let the parent answer a PR-Review ask that never reached it") is an ancestor of the merge base c3b4dad55, so it delivered G2 before this run. Its `AnswerParentAskOrUnreached` answers a latest PR-Review ask that ended no-parent or undelivered. The diff adds no `--head` flag, as A1 requires.
- G3: no change, as A1 directs, so no finding.
- A1: honored. The range contains no live-pane test and no `--head` flag, and makes no G2 change.

**Findings**
None. Every change traces to G1 or follows from it, and no decision left in scope goes unbuilt.

Tool calls:
1. `git merge-base --is-ancestor 3e8a5eebf c3b4dad55…`: confirmed that the commit delivering G2 precedes the review range. My context showed the commit's stat but not where it sits relative to the merge base.
