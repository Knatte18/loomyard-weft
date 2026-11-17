---
round: 1
ledger:
  - key: prepare-workdir-unresolved-anchor
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: overview-codex-launch-docs
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: unresolvable-config-dir-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: heddle-override-claudemd-imports-unread
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unread-scan-walks-whole-checkout
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: added-failure-paths-no-rows
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: subagent-terms-pins-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-testtiming-keep-markers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: set-help-drops-clauses
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings and no earlier ledger exists.
F1 (`prepare-workdir-unresolved-anchor`) is the one gating finding.
The directory-skip fix for the unread scan (F5) adds a constant and a skip rule, so it is marked `design-wrong` and needs a fresh look.
