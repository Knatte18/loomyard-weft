---
round: 1
ledger:
  - key: runprofile-load-third-stance-not-in-pattern
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: profile-check-untested-on-upsert-batch-and-merge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: running-pair-profile-notice-and-unchanged-rewrite-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: layered-document-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: setentries-local-layer-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings.
F1 (BLOCKING) is a PATTERN gap for `runprofile.Load`.
F2 is the gating MEDIUM: the profile check on `upsert-batch` and `merge` has no test row.
F3 to F5 are LOW test-coverage gaps.
All five are open and have local, text-determined fixes.
