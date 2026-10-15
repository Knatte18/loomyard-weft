---
round: 1
ledger:
  - key: card6-menu-hub-write-guard-subtest-unspecified
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card5-errAborted-comment-false-for-unchanged-invalid-abort
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-editLocal-abort-arm-coverage-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; there was no prior ledger.
F1 (card 6 leaves the real-hub menu write-guard subtest without a replacement) maps to the first key.
F2 (card 5's `ErrAborted` comment would be false for the unchanged-invalid-YAML abort) maps to the second.
F3 (card 5's claim that a test reaches `editLocal`'s abort arm) maps to the third.
All three are local corrections whose text the finding fixes, so each is `mechanical`.
