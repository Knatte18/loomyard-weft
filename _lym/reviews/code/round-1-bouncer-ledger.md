---
round: 1
ledger:
  - key: pair-lookup-ignores-other-machine-prefix
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: slugofbranch-strips-only-current-prefix
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: other-personal-keys-review-declined-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised three findings.
F1: origin lookup and `Add` with `AdoptOnly` try only the current machine's prefix, so a pair made under another prefix is not re-adopted.
F2: `slugOfBranch` derives a wrong slug and teardown lock path from another machine's branch.
F3: the second board goal point, reviewing the other personal keys, is declined without a recorded decision; deferred to the parent.
