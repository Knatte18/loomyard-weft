---
round: 2
focus:
  - "internal/gatecli/failurereport.go, renderFailureReport: past the line cap, are lint findings cut silently, and does the cut line name which runs lost lines and say that a lint step has no log, so its findings exist nowhere else? Check that every failed run keeps a place in the report."
  - "internal/gatecli/failurereport.go, renderFailureReport: does a run that has both a read failure and cause lines (timeout, no tests ran) still print the cause lines? Settle whether the new rows pin this."
  - "internal/gatecli/failurereport.go, qualify: does a compile error line from a nested module come out worktree-relative, whatever form the compiler prints the path in?"
  - "internal/gatecli build step (build.go) and run.go lint loop: do call-level tests cover a failing shared build listed in the failure report, a failing tier plus a failing lint keeping the tier's exit code, and a halted call printing the report before the closing line with no lint run?"
  - "internal/gatecli/failurereport.go, touchedFiles and runLints: do tests cover the hub branch (merge base with the pair's parent and the fall back to HEAD on an error) and the pattern lint branch, using the repo's hubforge fixture per PATTERN-test-isolation?"
  - "internal/gatecli/detachfront.go, front.replay: does a detached-front test pin the switch to replayStdout for a stdout past replayStdoutBytes? Also check that the reworded docs in internal/loomcli/lintcomments.go, internal/loomcli/lintpattern.go and contracts/specs/loom-plan-spec.md match the card gate's one call."
---

Focus on the code the fixes changed and the tests they added.
