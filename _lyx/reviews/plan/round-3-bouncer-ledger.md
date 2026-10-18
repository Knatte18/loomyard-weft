---
round: 3
ledger:
  - key: pattern-headroom-overshoot
    rounds: [1]
    status: resolved
  - key: card1-fake-event-flag-breaks-tests
    rounds: [1]
    status: resolved
  - key: card1-shuttlefake-default-not-inert
    rounds: [1]
    status: resolved
  - key: watcher-failed-call-double-send
    rounds: [1]
    status: resolved
  - key: card4-inject-timeout-unchanged-claim
    rounds: [1]
    status: resolved
  - key: verify-misses-internal-pattern
    rounds: [1]
    status: resolved
  - key: orch-start-withdraw-way-forward-gaps
    rounds: [1]
    status: resolved
  - key: orch-hold-refusal-no-way-forward
    rounds: [1]
    status: resolved
  - key: creel-failure-stuck-never-clears
    rounds: [1]
    status: resolved
  - key: outside-pane-protocol-row-linux-only
    rounds: [1]
    status: resolved
  - key: card2-orch-guard-removal-no-bound
    rounds: [1]
    status: resolved
  - key: card2-misses-broken-creelengine-tests
    rounds: [2]
    status: resolved
  - key: card11-misses-start-ensure-test
    rounds: [2]
    status: resolved
  - key: sandbox-creel-suite-c9-stale
    rounds: [2]
    status: resolved
  - key: plan-generation-cut-drops-qualifier
    rounds: [2]
    status: resolved
  - key: card1-signalengine-untested-doc-untargeted
    rounds: [2]
    status: resolved
  - key: restart-adopts-pending-test-redundant
    rounds: [2]
    status: resolved
  - key: stall-reason-no-clearing-rule
    rounds: [2]
    status: resolved
  - key: fresh-launch-stuck-text-no-clearing
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: inject-outcome-sending-record-ambiguous
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: old-state-stuck-text-survives-upgrade
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: new-watcher-tests-duplicate-kept-rows
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: command-inject-typing-order-unasserted
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

All round-1 and round-2 keys are resolved: round 3's fresh review raises none of them again.
Round 3 raised five new findings, none BLOCKING.
F1 maps to `fresh-launch-stuck-text-no-clearing`, F2 to `inject-outcome-sending-record-ambiguous`, F3 to `old-state-stuck-text-survives-upgrade`, F4 to `new-watcher-tests-duplicate-kept-rows`, F5 to `command-inject-typing-order-unasserted`.
F1 and F2 are the gating `design` ones, both MEDIUM and first raised this round; their fixes are local text and test-row additions the finding text already specifies.
