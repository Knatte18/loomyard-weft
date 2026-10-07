---
round: 1
ledger:
  - key: round-gate-tagged-compile-gap
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: publish-verify-unbounded-config
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: lyx-guard-missed-marker-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: comment-lint-flags-untouched-lines
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: comment-lint-terminator-rules
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: diff-mapping-rename-delete-baseless
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: empty-set-no-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: verify-record-retention-and-lookup
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: round-command-length-growth
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: tier-green-ordering-misplaced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: decision-record-narrative-duplicates
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review of the discussion pair, so no finding recurs.
Gating: `publish-verify-unbounded-config` (F2) and `lyx-guard-missed-marker-unbounded` (F3), both design at MEDIUM.
The other MEDIUM findings are consistency: the round gate compiling only the `integration` tag (F1) and the comment lint flagging untouched lines of a touched block (F4).
The remaining findings are LOW: lint terminators (F5), diff mapping (F6), the empty-set criterion (F7), verify-record retention (F8), command length (F9), the tier-green ordering (F10) and narrative duplicates (F11).
