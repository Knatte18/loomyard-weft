---
round: 2
focus:
  - "internal/testkit/tmuxkit/tmuxkit.go, Main: does a test pin the sequence snapshot before the run, run, check afterwards, exit code propagated, on both the Windows and Linux paths? Settle whether any shared sequence function is exercised with a fake run that creates a .git and one that creates none."
  - "internal/testkit/tmuxkit/tmuxkit.go, checkNewGitEntries: does each outcome (entry already removed, empty directory that cannot be removed, non-directory or non-empty entry) print its own accurate message, and does the exit code stay 1 in each? Is message order deterministic when both watched directories hold a new .git?"
  - "internal/testkit/tmuxkit/tmuxkit_test.go, TestCheckNewGitEntries: does some row seed the first watched directory and another the second, and does a row with a symlink named .git pin that the link is kept? Check any skip for Windows follows PATTERN-test-speed."
---

Round 1 findings in the focus entries above are code-level; the decision-class departure is the parent's to record and is not a focus subject.
