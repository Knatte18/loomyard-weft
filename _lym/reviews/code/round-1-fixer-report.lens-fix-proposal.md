```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out (fix-proposal lens).

**MEDIUM: the F3 skip in `PlaySession` (`internal/testkit/tmuxrec/session.go`, `t.Skipf` on `rec.Source.OS != runtime.GOOS`)**

- **Summary:** the proposed OS skip is a skip by design, and it drops far more than the replay tests.
- **Rule:** `PATTERN-test-speed`, "No test skips by design", says "A test that means something on one OS alone sits in a file only that OS compiles" and "A table row or subtest bound to one OS is not generated on another". It allows a remaining skip only for a prerequisite the host lacks, "such as a binary, a privilege or a non-root user".
- **Reach:** `newPRReviewFixture` (`internal/landingshed/prreview_test.go:727`) calls `newPRSessionWorld` for every fixture. On Windows or macOS, `TestPRReview_Call`, `TestPRReview_Wait`, `TestPRReview_Session` and the other fixture-based PR-Review tests therefore all skip, including rows that never touch a session, such as pull request creation, push and mode handling.
- **macOS:** the skip fires there too, although the OS branch the finding cites (`listSessionsArgv`) treats darwin like Linux. I did not check whether the Linux recording would in fact replay on darwin.
- **Second rule:** `PATTERN-test-stand-ins` wants the reading code to run "over every recording in a `tier0` test", which no longer happens off Linux.
- **Recorded so far:** the `Loosened-Check` trailers record the loss but do not resolve the conflict with the rule. The fixer-report should at least name it, with the alternative considered: a build constraint would require moving the fixtures `finalize_pr_test.go` shares.

I found no other concrete breakage in the proposals carried out for F1, F2, F4, F6, F7, F9, F10, F11, F12, F13, F14 or F15, or in the name-length and fixed-name check from F8.

Tool calls: none.
