---
round: 1
ledger:
  - key: gate-passed-inside-outer-lock
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dobby-goal-point-unbuilt-false-premise
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: four-gate-sites-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: removed-flag-rows-no-loosened-check
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: deferral-doc-overstates-when-it-fires
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: coalesceflagged-admission-release-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: revive-skip-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: errnosession-narrow-match-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pauserecord-doc-one-lock-hold-operation
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-omits-skipped-part
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: revive-docs-omit-skip
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: status-help-omits-prime
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings.
The gating pair is F1, where the pause gate sits inside the board write lock and the pair's teardown lock, and F3, where four gate sites have no test.
F2 asks for the dobby spawn path of the incident to be found or the summary corrected.
F4 asks for a `Loosened-Check:` trailer for the dropped flag rows.
F5 to F12 are doc and test-coverage corrections.
