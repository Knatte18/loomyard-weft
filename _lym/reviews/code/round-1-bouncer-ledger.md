---
round: 1
ledger:
  - key: delete-before-preflight-refusals
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tracked-file-refusal-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: errtaskpathtaken-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: errstraypathkept-add-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: operational-failures-replace-refusal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failure-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings: F1 (delete-before-preflight-refusals), F2 (tracked-file-refusal-unpinned), F3 (errtaskpathtaken-text-stale), F4 (errstraypathkept-add-undocumented), F5 (operational-failures-replace-refusal) and F6 (failure-branches-untested).
F1 and F2 are the gating pair.
Each fix text already fixes its result, so all are marked mechanical.
There was no earlier ledger.
