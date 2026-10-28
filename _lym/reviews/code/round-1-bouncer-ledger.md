---
round: 1
ledger:
  - key: branch-verbose-wrongly-list-mode
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: branch-value-skip-and-trailing-dashdash-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised two findings.
F1 (`branch-verbose-wrongly-list-mode`): `-v` and `--verbose` are classed as list mode in the fork audit's `git branch` classifier, so `git branch -v foo` passes although git creates the branch.
The fix is local and fully specified by the finding text.
F2 (`branch-value-skip-and-trailing-dashdash-unpinned`): missing test rows for the separate-word value skip and the trailing `--` branch.
