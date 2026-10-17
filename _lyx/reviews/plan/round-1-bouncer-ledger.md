---
round: 1
ledger:
  - key: pattern-headroom-overshoot
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card1-fake-event-flag-breaks-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card1-shuttlefake-default-not-inert
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: watcher-failed-call-double-send
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card4-inject-timeout-unchanged-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-misses-internal-pattern
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: orch-start-withdraw-way-forward-gaps
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: orch-hold-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: creel-failure-stuck-never-clears
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: outside-pane-protocol-row-linux-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card2-orch-guard-removal-no-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings: one BLOCKING and three MEDIUM design findings gate, the rest are LOW.
The BLOCKING one is the overview headroom overshoot from the two declared PATTERN rewrites.
The MEDIUM ones are card 1's two test-kit side effects and the double-send hole in the live watcher after a failed creel call.
Nothing is resolved yet; the first round has no earlier ledger.
Finding F1 maps to `pattern-headroom-overshoot`, F2 to `card1-fake-event-flag-breaks-tests`, F3 to `card1-shuttlefake-default-not-inert`, F4 to `watcher-failed-call-double-send`, and F5 to F11 in review order to the remaining keys.
