---
round: 1
ledger:
  - key: lymtemp-link-safety-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lymtemp-prepass-failure-aborts-removal
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: permkit-stdlib-only-unenforced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: logger-delete-failure-tolerance-keep-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 (`lymtemp-link-safety-untested`) is the gating one: the sweep's never-follow-a-link claim has no test row.
F2 asks to make the owner-writable pre-pass best-effort, which changes the pass's contract.
F3 asks for a permkit stdlib-only enforcement test, and F4 for a `//testtiming:keep` mark.
