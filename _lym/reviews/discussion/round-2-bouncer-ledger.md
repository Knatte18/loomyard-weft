---
round: 2
ledger:
  - key: d8-agentconfig-premise-false
    rounds: [1]
    status: resolved
  - key: live-gate-names-tier-not-key
    rounds: [1]
    status: resolved
  - key: sandbox-reaper-outside-runproc
    rounds: [1]
    status: resolved
  - key: copies-cannot-get-slots-in-live-gate
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: step1-tier2-run-not-serial
    rounds: [1]
    status: resolved
  - key: model-rotation-makes-converged-unreachable
    rounds: [1]
    status: resolved
  - key: red-live-gate-deferred-cause-no-outcome
    rounds: [1]
    status: resolved
  - key: stray-report-writers-archive-order
    rounds: [1]
    status: resolved
  - key: d10-loosened-order-clause-no-reach
    rounds: [1]
    status: resolved
  - key: treadle-deletion-needs-parent
    rounds: [1]
    status: resolved
  - key: d5-reap-bounds-miss-cases
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: lightweight-budget-reader-ignores-hardener-yaml
    rounds: [1]
    status: resolved
  - key: harden-fix-verify-source-ambiguous
    rounds: [1]
    status: resolved
  - key: code-facts-unpinned
    rounds: [1]
    status: resolved
  - key: hardener-yaml-keys-unclassified-run-settings-override
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: live-gate-can-run-no-test-and-read-green
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: treadle-settled-question-stale-in-open-risks
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised 14 findings, three BLOCKING; round 2 judged the fixes and raised five.
Eleven of the fourteen round-1 keys are resolved: D8's `agentconfig` premise, the live gate's gate keys, the runproc reaper, the serial claim, model rotation, the red-live-gate outcome, the stray-report order, D10's order clause, the treadle question, the lightweight budget reader, the fix verify source and the unpinned code facts.
`copies-cannot-get-slots-in-live-gate` stays open as F18: the call's n-slot lease and its own tier run contend for the same pool.
`d5-reap-bounds-miss-cases` stays open as F19, LOW: `internal/pairteardown` runs no sandbox reap for an operator removal.
Three keys are new this round and BLOCKING: D9's unclassified `hardener.yaml` keys with an unbounded `auto_accept`, a live gate that can start no test binary and read green, and the settled treadle question left in Open risks and D8's rationale.
