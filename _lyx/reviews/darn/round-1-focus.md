---
round: 1
focus:
  - "internal/reedengine/window.go read-back: does a reply from the new window itself that carries a value other than `off` still kill it and wrap ErrWindowReadBack, and does the split on the first space hold for every reply shape tmux can give (empty id, no value, trailing newline)?"
  - "internal/reedengine/window_test.go and windoworder_test.go: do the fake answers and the new global-scope row cover the id-mismatch branch and the genuine-mismatch branch, or does an existing test already cover either?"
  - "internal/reedengine/doc.go and _lyx/landing/summary.md: does each claim about tmux's answer for a vanished target and about what was run match the diff?"
---

Seed focus for round 1.
The change is small and confined to `internal/reedengine`, so the focus is the branch logic and the test coverage of each branch.
