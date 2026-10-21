---
round: 1
ledger:
  - key: verify-runner-unslotted
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: none-scoper-floor-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: darn-tree-scope-landing-lost
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: empty-tree-targets-widen
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slot-wait-inside-timeout
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: targetless-tier-whole-vs-skipped
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-bound-false-under-drop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: key-by-key-set-bootstrap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-step-no-owning-tier
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: confirm-fix-renamed-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: test-holding-definition
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: build-vet-lose-p-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: non-gate-slot-holders
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-in-base-command
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dash-c-computed-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: guard-tier-refusal-order
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-bound-which-tiers-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d10-bound-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: compound-tree-scope-command
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drain-deploy-transition
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-machine-tier-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: watchdog-vstest-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: webster-prompts-render-verify
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 23 findings, all open.
The two BLOCKING ones are the unslotted card `Verify:` runner and the `none`-scoper floor contradiction.
The MEDIUM design cluster covers tree-scope and darn paths, slot wait inside caller timeouts, the guard step's owning tier, confirm-the-fix on a renamed test, `-p` cap loss and non-tier slot holders.
The rest are LOW bound and wording corrections.
