---
round: 2
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
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card11-misses-start-ensure-test
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: sandbox-creel-suite-c9-stale
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: plan-generation-cut-drops-qualifier
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card1-signalengine-untested-doc-untargeted
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: restart-adopts-pending-test-redundant
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stall-reason-no-clearing-rule
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1's eleven findings no longer apply: round 2's fresh review settles the headroom, the double send, the creel-failure clearing and the bounds, and honors the kit's inert default.
Round 2 raised seven new findings, none `design`.
F1 maps to `card2-misses-broken-creelengine-tests`, F2 to `card11-misses-start-ensure-test`, F3 to `sandbox-creel-suite-c9-stale`, F4 to `plan-generation-cut-drops-qualifier`, F5 to `card1-signalengine-untested-doc-untargeted`, F6 to `restart-adopts-pending-test-redundant`, F7 to `stall-reason-no-clearing-rule`.
The two BLOCKING ones are missing Edit targets for tests that the cards' own changes break.
