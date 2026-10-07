---
round: 2
focus:
  - "Comment line breaks: in the branch's added or changed Go comments, including `internal/battenshed/innerrun.go`, `internal/battenshed/innerrun_test.go`, `internal/reedengine/lifecycle.go`, `internal/reedengine/windowsize.go`, `internal/reedcli/resume.go`, `internal/tokenvocab/doc.go`, `internal/tokenvocab/tokenvocab_test.go` and `internal/reedengine/waitmark_integration_test.go`, does every paragraph keep one sentence per line, with no mid-sentence column wrap and no two sentences on one line, as PATTERN-comment-line-breaks requires?"
  - "Test parallelism: do the tests the branch adds in `internal/loomcli`, `internal/battencli`, `internal/reedengine`, `internal/battenshed` and `internal/shuttleengine` call `t.Parallel` unless they touch process-global state, and where a test swaps the logger output, does a comment at the test or atop its file name that state, as PATTERN-test-speed requires?"
  - "Stuck reason of the websterengine verify gate (`verifyGateStuckReason` in `internal/websterengine/verifygate.go`): when the gate exhausts its attempts on a timed-out verify report, which carries `TimedOut` and a log path but no `Failures` or `Dirty`, does the stuck reason name the timeout, and does a test reach that case?"
  - "Revive episodes in `internal/battenshed/innerrun.go` (`callHalted`, `reviveDeadDriver`): when a child resumes and re-halts between two polls while its driver is dead, is the revive attempted again for the new halt episode, and does a test cover it?"
  - "Redundancy in the branch's `internal/reedengine` strand tests: does any new test repeat a case that a public-surface test such as `AddStrandUnless` already asserts, and does every `//testtiming:keep` reason still hold?"
  - "Markdown line breaks: do the lines the branch changed in `docs/overview.md` and any other `.md` file keep one sentence per line, as the repository's `CLAUDE.md` requires?"
---

Every entry names code or docs the branch changed, so each can be checked against the committed diff without any prior review.
