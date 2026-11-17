---
round: 2
focus:
  - "In internal/raddleparser/check.go, `vocabulary()` must refuse a part whose `labels` list one module label more than once, with a way-forward clause in its Detail. Settle whether every duplicate the board's `validateLabels` in internal/boardengine/labels.go refuses is also refused at the proposal check, and whether contracts/specs/raddle-proposal-spec.md lists the check."
  - "In internal/raddleparser/check_test.go, `TestCheck` must pin the duplicate-label refusal. Settle whether the row sets a repeated label on a part and wants only the duplicate check id."
---

Both entries follow the fix for the one round-1 finding, which no fresh reviewer has seen.
