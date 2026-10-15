---
round: 1
ledger:
  - key: opening-paragraph-unconditioned-rules-read
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: verify-skips-tools-sandbox-package
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1: card 1's opening paragraph tells the Master to read the repo's rules unconditionally, against the vacuous-when-empty rule.
F2: card 1 edits an embedded file of `tools/sandbox`, which the `## verify:` section neither vets nor tests.
Both carry an exact fix in the review.
