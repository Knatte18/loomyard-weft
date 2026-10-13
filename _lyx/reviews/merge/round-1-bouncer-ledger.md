---
round: 1
ledger:
  - key: probequiet-ignores-go-driver-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding.
`probeQuiet` in `internal/pairteardown/teardown.go` does not probe the parent's `shedrun.DriverLock`, so a settling Go driver reads as quiet and `fabric remove` can remove a worktree under it.
The review's fix text names the exact change and test row, so it is a local correction.
