---
round: 2
ledger:
  - key: d5-nested-module-caller-coverage-unbounded
    rounds: [1]
    status: resolved
  - key: d9-fix-shape-suppresses-real-defect
    rounds: [1]
    status: resolved
  - key: d1-interface-dispatch-drop-unbounded
    rounds: [1]
    status: resolved
  - key: d6-rename-of-gomod-ignored
    rounds: [1]
    status: resolved
  - key: requirements-only-in-open-risks
    rounds: [1]
    status: resolved
  - key: d8-go-dash-C-base-after-cd
    rounds: [1]
    status: resolved
  - key: d6-move-of-gomod-uncounted
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-dispute-route-forbidden-by-fix-rule
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-bound-overstates-card-gate
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, two BLOCKING design.
Round 2 found none of them still standing; the review confirms D5's nested-module scan, D9's keep-the-defect shape and D6's `Rename` handling.
Round 2 raised three new findings: D6/D8 miss a `go.mod` `Move` (MEDIUM design), D9's rationale cites a dispute route the fix rule forbids (MEDIUM design), and D1's bound overstates what the card gate builds (LOW).
The two MEDIUM design findings are first raised in round 2 and are carried into Open risks.
