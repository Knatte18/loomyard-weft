```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the decision record's goal points (code-scope lens).

**Decisions.** The record is a decision record alone. `## Goal` holds one goal point, and the record has no `Added after Discussion` entries.
- **G1:** the PR-Review row detects a dead PR-Review session. It then either re-spawns the session under the interrupt policy or halts with a stop the parent can act on (`_lym/discussion/decision-record.md:6`).

**Changes, each traced to a decision:**
- **C1 — dead-session halt in the wait.** New `haltOnEndedSession`, called on a `!polled` wake and on a judged-open wake that did not stop and resume the session (`internal/landingshed/prreview.go:705-707`, `:735-737`, `:744-760`). Traces to G1, through its halt branch. The halt reason names `lym loom resume`, which gives the parent a way forward.
- **C2 — `keepsSession` threaded into the wait.** It is passed through `wait`'s parameter, the `prWaitState` field and the three `openPullRequest` call sites (`prreview.go:306`, `:340-346`, `:371`, `:596-597`, `:618`, `:625`). Forced by C1: it limits the check to waits whose call started or reused the session, so a wait on an ended review or a merged pull request is not halted on a stale strand.
- **C3 — doc comments.** `doc.go:24`, the `PRReview` type doc (`prreview.go:49`), the `wait` doc (`:614-616`) and the `wake` doc (`:660`). Forced by C1 and C2: they document the new halt and the new parameter.
- **C4 — tests.** Two new `TestPRReview_Wait` rows (`prreview_wait_test.go:994-1020`) and the extra `false` argument at `prreview_wait_test.go:1131`. Forced by C1 and C2: coverage of the halt, and the caller update the new signature requires.

**Changes per decision:**
- G1 is built by C1, with C2, C3 and C4 forced by C1.

**Findings:** none.
- Every hunk traces to G1 or is an edit G1's change forces.
- G1 is built through its halt option. The board entry offers re-spawn or halt as alternatives, so leaving out re-spawn is no scope gap.
- No commit in the range carries a `Pattern-Departure:` trailer, and none is needed.

Tool calls: none. The inherited context held the diff, the decision record and the commit log.
