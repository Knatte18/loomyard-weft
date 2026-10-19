---
round: 1
ledger:
  - key: gitexec-probe-unlogged-stale-exemption
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-skip-drops-empty-from-start-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-tmux-34-session-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-marker-does-not-identify-probe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-quarry-gitsrc-bypasses-floor-check
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-process-cache-vs-per-test-hooks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-acceptance-criterion-tmux-floor
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d7-criterion-needs-undeclared-seam
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: d3-attach-repin-unmarked-prime
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: rejected-alternative-and-history-prose
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: apple-git-in-admitted-set
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review of the discussion pair; the ledger has no earlier entries to carry forward.
The gating findings centre on D4's version-floor checks (logging exemption, the quarry git path, cache scope in tests) and on D2, D3 and D6 making claims their mechanisms cannot keep.
The remaining findings are missing acceptance coverage, D7's seam, one D3 marking gap and two NITs.
