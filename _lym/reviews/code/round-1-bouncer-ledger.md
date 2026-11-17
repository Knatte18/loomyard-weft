---
round: 1
ledger:
  - key: reinterpret-drops-nonzero-count
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: repeat-window-pruning-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hookedit-help-old-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: editrecorder-doc-omits-hash
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dropeditloopedits-null-and-nonmapping-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-line-empty-value
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: recordedit-test-dead-carried-check
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings.
The gating pair is F1, where the reinterpretation drops a non-zero `edit_loop_edits` instead of flagging it, and F2, where no test pins the expiry of a repeat from the window.
F1's fix changes the reinterpretation's contract, so its new text needs a fresh review.
The other findings are local corrections whose text the findings fix.
