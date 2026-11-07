---
round: 1
ledger:
  - key: seam-installed-before-template-copies
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `seam-installed-before-template-copies`.
`TestWorktreeSeam_Events` installs the worktree seam at its top level, and later subtests take template copies through `newOriginHub` and `newMovesPair`.
The review's fix text names the copies to move before the install, so the fix is a local correction.
