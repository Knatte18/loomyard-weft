---
round: 1
ledger:
  - key: unrelated-pattern-entries-rewritten
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: fan-list-null-entry-refused
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-loomsegmentperches-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: untested-setting-refusals
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: fixfankey-carries-citation
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings.
F1 (BLOCKING) is a commit that shortened PATTERN entries beyond the one declared change.
F2 to F5 are LOW: a null fan-list entry refused, a stale comment, untested refusals, and a citation stored in `FixFanKey`.
