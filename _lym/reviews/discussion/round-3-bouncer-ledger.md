---
round: 3
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
    status: resolved
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
    status: resolved
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
    status: resolved
  - key: live-gate-can-run-no-test-and-read-green
    rounds: [2]
    status: resolved
  - key: treadle-settled-question-stale-in-open-risks
    rounds: [2]
    status: resolved
---

Round 1 raised 14 findings and round 2 five; round 3 raised none.
The five keys open after round 2 are resolved by the round-3 review:
D9 classifies every `hardener.yaml` key, the live gate reads red when it starts no test binary, D8 cites the operator's treadle entry, D4 states the `--copies` slot rules, and D5 has `internal/pairteardown` clear the sandbox on every teardown route.
No key is open.
