---
round: 1
ledger:
  - key: dropped-archive-tag-tests
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised one finding: seven dropped tests in `internal/fabricengine` pin archive tags, which the decision record's keep rule says to retain with a `testtiming:keep` reason.
The review's fix text names each test and the restore source, so the correction is local.
