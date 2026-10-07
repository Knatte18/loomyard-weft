---
round: 1
ledger:
  - key: comment-fixed-column-wrapping
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: regression330-test-no-longer-pins-behavior
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: added-behaviors-without-coverage
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: duplicated-batchifier-failure-refusal-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: cost-test-helpers-missing-doc-comments
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised five findings, none recurring.
The BLOCKING one is comment wrapping across batcher, websterengine, webstercli, gitrepo and `tools/tokencount`.
The gating design finding is the rewritten Regression330 test, which no longer pins the accept-after-begin-batch-rewrite path.
The remaining three are a coverage gap on three added behaviors, a duplicated refusal message, and two undocumented cost test helpers.
