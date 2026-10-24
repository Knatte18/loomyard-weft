---
round: 1
ledger:
  - key: from-unknown-key-not-refused
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: from-trailing-data-more-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: amendment-restore-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: discussionparser-doc-omits-appenddecisions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: add-help-required-flags-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: appenddecisions-comment-references-sibling
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, mapped as F1 to `from-unknown-key-not-refused`, F2 to `from-trailing-data-more-check`, F3 to `amendment-restore-untested`, F4 to `discussionparser-doc-omits-appenddecisions`, F5 to `add-help-required-flags-contradiction` and F6 to `appenddecisions-comment-references-sibling`.
F1 gates: the decision record requires refusing an unknown key in the batch file, and the diff does not.
The rest are local corrections the review text already fixes.
