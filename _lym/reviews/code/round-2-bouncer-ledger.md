---
round: 2
ledger:
  - key: turn-on-remote-control-untested
    rounds: [1]
    status: resolved
  - key: rc-click-on-already-on-session
    rounds: [1]
    status: resolved
  - key: launch-branches-no-test-rows
    rounds: [1]
    status: resolved
  - key: rc-bar-fallbacks-no-test-rows
    rounds: [1]
    status: resolved
  - key: duplicate-orch-strands-misreported
    rounds: [1]
    status: resolved
  - key: adopted-strand-loses-lit-state
    rounds: [1]
    status: resolved
  - key: record-write-failure-reported-as-failed-click
    rounds: [1]
    status: resolved
  - key: codex-orch-draws-rc-button
    rounds: [1]
    status: resolved
  - key: orch-group-help-stale
    rounds: [1]
    status: resolved
  - key: contained-reed-tests-need-keep-marks
    rounds: [1]
    status: resolved
  - key: rc-verb-names-repeated-as-literals
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: summary-contradicts-click-codex-button
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: already-running-adopt-record-untested
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised ten findings, all absent from round 2 and so resolved.
Round 2 raised three new ones.
F1 is BLOCKING: the `rc-on` and `rc-bar` verb names are declared in orchcli but repeated as literals in reedengine's composed lines.
F2 is a change description that contradicts the diff on the click, Codex and the button's visibility.
F3 is a missing assertion on the already-running adopt branch's record write.
