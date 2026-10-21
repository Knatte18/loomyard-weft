---
round: 2
ledger:
  - key: verify-runner-unslotted
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: none-scoper-floor-contradiction
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: non-gate-slot-holders
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: confirm-fix-not-one-gate-call
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: flow-mapping-values-unquoted
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: in-flight-verify-lines-risk
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: darn-tree-scope-landing-lost
    rounds: [1]
    status: resolved
  - key: empty-tree-targets-widen
    rounds: [1]
    status: resolved
  - key: slot-wait-inside-timeout
    rounds: [1]
    status: resolved
  - key: targetless-tier-whole-vs-skipped
    rounds: [1]
    status: resolved
  - key: d3-bound-false-under-drop
    rounds: [1]
    status: resolved
  - key: key-by-key-set-bootstrap
    rounds: [1]
    status: resolved
  - key: guard-step-no-owning-tier
    rounds: [1]
    status: resolved
  - key: confirm-fix-renamed-test
    rounds: [1]
    status: resolved
  - key: test-holding-definition
    rounds: [1]
    status: resolved
  - key: build-vet-lose-p-cap
    rounds: [1]
    status: resolved
  - key: merge-in-base-command
    rounds: [1]
    status: resolved
  - key: dash-c-computed-scope
    rounds: [1]
    status: resolved
  - key: guard-tier-refusal-order
    rounds: [1]
    status: resolved
  - key: d2-bound-which-tiers-run
    rounds: [1]
    status: resolved
  - key: d10-bound-incomplete
    rounds: [1]
    status: resolved
  - key: compound-tree-scope-command
    rounds: [1]
    status: resolved
  - key: drain-deploy-transition
    rounds: [1]
    status: resolved
  - key: stale-machine-tier-entry
    rounds: [1]
    status: resolved
  - key: watchdog-vstest-claim
    rounds: [1]
    status: resolved
  - key: webster-prompts-render-verify
    rounds: [1]
    status: resolved
---

Round 1 raised 23 findings; round 2's fresh review raised 6 and none of the other round 1 keys, so those are resolved.
Three keys carry over: the unslotted card `Verify:` runner (now the fork's own run of the line, BLOCKING), the `none`-scoper contradiction (now an acceptance criterion against D3), and non-tier slot holders (now `cmd/testtiming`'s modes).
New in round 2: the confirm-the-fix pass as raw `go test`, unquoted braces in flow-mapping values, and in-flight plans whose Verify lines hold `--tags`.
