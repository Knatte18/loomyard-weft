---
round: 1
ledger:
  - key: drift-child-carries-shell-cd-diagnosis
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: way-forward-cd-path-unquoted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: subpath-anchor-cd-target-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shipped-entry-overclaims-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pinned-row-asserts-heading-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: blast-radius-omits-agent-instructions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-test-doc-omits-message-pin
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised only LOW and NIT findings, so the loop converged on its first round.
The code findings concern the new gate refusal text (where it surfaces, its quoting, its test coverage) and the shipped `lym-shell-directory` entry.
The rest concern the change description and a test doc comment.
Whether `env -C` works outside Linux and Git Bash was left unsettled, with no finding filed.
