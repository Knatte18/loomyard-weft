---
round: 1
ledger:
  - key: default-run-selection-undecided
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: repeat-count-vs-fork-replay-skip
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: vanished-session-rule-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: read-only-bound-not-by-type
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: no-cutoff-header-claude-term
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 opened five findings.
The default run selection (no slugs) has no decision for candidate discovery or the recency key once the projects root leaves `tokencount`.
D7's vanished-session rule leaves an unbounded silent undercount and names false causes.
D9's repeated-lines figure conflicts with the fork replay skip, D6's read-only bound does not hold by type, and the `-no-cutoff` header keeps a Claude layout term.
