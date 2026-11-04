---
round: 1
ledger:
  - key: webster-verify-gate-ignores-runner-crash
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: gatecli-closing-line-exitcode-detach-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verifytree-crash-line-reading-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: landingshed-parentalign-fixerend-crash-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: loomshed-terminal-on-crash-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retrytier-shortage-then-crash-mislabelled
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: exitcodeof-build-signal-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prreview-gate-no-halt-on-repeat-crash
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised eight findings and no earlier ledger exists.
The BLOCKING finding is the webster verify gate and `lym webster verify` still treating a repeat runner crash as a failure to fix.
Four MEDIUM design findings are missing tests for the crash paths in `gatecli`, `verifytree`, `landingshed` and `loomshed`.
Three LOW findings cover the shortage-then-crash label, the build exit code and the pr-review gate's missing halt.
