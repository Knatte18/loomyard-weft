---
round: 1
ledger:
  - key: reedengine-doc-stale-moved-test-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-md-test-speed-line-omits-no-skip-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 was the first round.
F1 maps to `reedengine-doc-stale-moved-test-path`: the doc comment cites the old file of a moved test.
F2 maps to `pattern-md-test-speed-line-omits-no-skip-rule`: the rule lives only in the background file.
Both are local corrections whose text the finding fixes.
