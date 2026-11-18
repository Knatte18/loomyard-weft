---
round: 1
ledger:
  - key: replace-base-any-age-exempts-removal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: findings-path-repeats-logs-literal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: group-endpoints-follow-input-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: replaced-decision-record-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: grouping-by-reason-and-unkeyed-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cap-outside-budget-and-edges-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-halt-after-resume-unchecked
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: before-session-dating-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreturned-session-halt-repeats-first-text
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: findings-file-checked-by-count-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guardchange-path-pinned-code-checkout-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, all class `design`.
F1 (`replace-base-any-age-exempts-removal`) is the one design-wrong fix: it changes which replace commit counts as a base.
F2 is a repeated path literal against `PATTERN-code-conventions`.
F3 is a correctness gap in group endpoints; F4, F5 and F6 to F11 are test and edge gaps with fixes the findings spell out.
Nothing is resolved yet and no earlier ledger exists.
