---
round: 1
ledger:
  - key: owed-code-tip-not-pushed-when-origin-lacks-code-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pushbeam-takeover-read-off
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: recorded-landing-commit-archived-merge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: keep-marker-recordslandingcommit-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: keep-marker-writesboardstatus-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 found five findings.
F1 (owed code tip with no origin code branch) and F3 (landing commit recorded before an archive reset) are gating design findings.
F2 (`PushBeam` without `Here` and `RunSlug`) is a scope finding.
F4 and F5 ask only for `//testtiming:keep` markers on two tests.
