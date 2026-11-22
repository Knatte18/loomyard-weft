---
round: 1
ledger:
  - key: joiner-test-seam-markers-incomplete
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: joiner-test-missing-testtiming-keep
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: seedless-run-widens-records-walk-to-full-history
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: history-walks-still-traverse-whole-branch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bound-changes-landing-join-results
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: records-walk-doubling-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: board-floor-older-time-join-opening-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tip-as-board-in-effect-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: widening-stop-at-earlier-tag-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: board-in-effect-assumes-committer-time-follows-ancestry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: doc-misplaces-first-records-walk-opening
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
Eleven findings are open: F1 and F2 (BLOCKING test-marker gaps), F3 to F7 (MEDIUM, covering the unbounded walks, the changed join results and unpinned behavior), F8 to F10 (LOW) and F11 (NIT doc).
F3, F4, F5 and F10 change an approach or contract and are marked `design-wrong`.
The rest are local corrections their fix text already fixes.
