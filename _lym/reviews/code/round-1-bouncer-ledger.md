---
round: 1
ledger:
  - key: adoptcli-doc-envelope-fields-missing
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: yamlengine-doc-restates-function-docs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: batcher-doc-restates-symbol-docs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: summary-doc-go-count-off-by-one
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
F1 (adoptcli envelope fields), F2 (yamlengine restatement), F3 (batcher restatement) and F4 (summary count) are all non-gating.
Each fix is a local doc or summary edit whose result the finding text already fixes.
