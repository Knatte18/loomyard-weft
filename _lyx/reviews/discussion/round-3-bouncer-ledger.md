---
round: 3
ledger:
  - key: selfexec-leaf-set-outside-pattern
    rounds: [1]
    status: resolved
  - key: replace-selection-moves-viewer-off-shell
    rounds: [1]
    status: resolved
  - key: hidden-window-insert-target-missing
    rounds: [1]
    status: resolved
  - key: no-shell-fallback-order-unbounded
    rounds: [1]
    status: resolved
  - key: migration-step-failure-bound-missing
    rounds: [1]
    status: resolved
  - key: stale-sidecar-consequence-only-in-support-log
    rounds: [1]
    status: resolved
  - key: scope-out-carries-history
    rounds: [1]
    status: resolved
  - key: migration-criterion-says-last
    rounds: [1]
    status: resolved
  - key: attach-selection-rides-suppressed-chain
    rounds: [2]
    status: resolved
  - key: migration-warn-recurs-each-open-and-attach
    rounds: [2, 3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: newest-agent-premise-unqualified-on-psmux
    rounds: [2]
    status: resolved
  - key: strand-window-fallback-renames-overview-or-agent
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: attach-select-moves-other-attached-clients
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-lowest-index-unconditional-on-psmux
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: migration-criterion-omits-selvage-alone-and-no-size
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 left eight resolved keys from round 1 and three from round 2.
Round 3 resolves the round 2 attach-selection and newest-agent findings; the review says the attach selection now rides every `AttachArgv` return.

The round 2 LOW key on D10's Warn recurrence returns as a BLOCKING consistency finding: D10's Warn-once-per-session with a never-cleared option departs from the `internal/logger` Level policy, which asks for Warn once per episode, Debug while it persists and Info at recovery.
Its fix is a local rewrite the finding text already supplies.

Two new `design` MEDIUM findings: the strands' window resolver can fall back to the overview or an agent window, so D10 can rename and move the wrong window; and the attach `select-window` moves every attached client.
Both change a decision and need a fresh review.
Two LOW findings are mechanical: D1's unconditional lowest-index claim on psmux, and the migration criterion's missing Selvage-alone and no-client-size paths.
