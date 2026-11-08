```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (F1 to F5) by building a case where it is wrong or breaks something else in the record.

All five fixes were carried out as proposed. I found one LOW finding and nothing that breaks.

- **LOW**, the D7 Rationale sentence "The build or test running when the budget runs out can add up to the longer of `redundancy_build_timeout_min` and `redundancy_test_timeout_sec` plus D5's grace."
  - **Summary:** the sentence can be read as adding the grace to the test timeout only. A timed-out build waits out the same grace, so the true overrun is the longer of the two timeouts, plus the grace.
  - **Evidence:** D5 says "a build past it is ended with its process group like a test". The grace wait follows that ending ("the verb then waits for the test's exit and output at most a fixed grace"). So a build that times out at the budget's end can also add the grace, and the sentence's wording leaves the grace off the build term.

Cases I built that did not hold:
- **F1:** with every tier carried, no slot is held before the step, and its own slot wait is stated as unbounded. A budget spent on the `lym` build leaves the remaining tests "not judged for the spent budget", which the rule already covers.
- **F2:** a concurrent manual `lym gate testmap` overwrites the exit record, so the dobby sees another process ID and treats the refresh as leaving none. That is the safe fallback the fix states. The leftovers wait for the next prime HEAD move.
- **F3:** a malformed keep marker found during the step becomes an error line, under D7's rule that every error is a report line, and that line already blocks the repeat-call skip.
- **F4:** a gitignored generated Go file in the worktree has no text at the merge base, so D7 step 6's drift line never names it. Aligning D4's set causes no new mismatch.
- **F5:** the prime worktree opens a full fabric handle (`fabricengine.Open` through `OpenCodeWorktree`). `Fabric.Status` includes untracked files, because go-git marks them as something other than Unmodified (`internal/gitrepo/worktree.go:32`). It also returns weft-side entries labelled `SideWeft`, so the verb keeps only the warp-side ones. The plan writer can see that from the type.

Tool calls:
1. Grepped `internal/fabricengine` and `internal/gitrepo/worktree.go` for how a Fabric is opened and how status is read, because my context lacked both.
2. Read `internal/fabricengine/codeworktrees.go:40-69` to confirm the prime worktree opens a full fabric handle, because my context did not show `OpenCodeWorktree`.
