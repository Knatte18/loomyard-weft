---
round: 1
ledger:
  - key: edit-directive-test-pins-stencil-prose
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised one finding, F1: the `scratchBuildBan` assertion in `internal/editdirective/editdirective_test.go` pins the shipped stencil's static prose, which `PATTERN-test-speed` forbids.
The fix deletes the const and its check and restores the test's doc comment.
