---
round: 2
ledger:
  - key: gate-passed-inside-outer-lock
    rounds: [1]
    status: resolved
  - key: dobby-goal-point-unbuilt-false-premise
    rounds: [1]
    status: resolved
  - key: four-gate-sites-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: removed-flag-rows-no-loosened-check
    rounds: [1]
    status: resolved
  - key: deferral-doc-overstates-when-it-fires
    rounds: [1]
    status: resolved
  - key: coalesceflagged-admission-release-untested
    rounds: [1]
    status: resolved
  - key: revive-skip-untested
    rounds: [1]
    status: resolved
  - key: errnosession-narrow-match-unpinned
    rounds: [1]
    status: resolved
  - key: pauserecord-doc-one-lock-hold-operation
    rounds: [1]
    status: resolved
  - key: overview-omits-skipped-part
    rounds: [1]
    status: resolved
  - key: revive-docs-omit-skip
    rounds: [1]
    status: resolved
  - key: status-help-omits-prime
    rounds: [1]
    status: resolved
---

Round 1 raised twelve findings, and round 2 resolved eleven of them.
The BLOCKING gate-inside-lock finding is fixed at every site.
Only `four-gate-sites-untested` remains, narrowed from four untested sites to the weft replay test, which does not pin `lockWeftForReplay`'s admission.
It is LOW, with a mechanical fix the review's text already fixes.
