---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: migration-warn-recurs-each-open-and-attach
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: newest-agent-premise-unqualified-on-psmux
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's eight findings are all resolved: round 2's focus verdicts settle the `internal/selfexec` leaf set, the explicit insert targets, the replace rule and the migration's race and failure handling, and the review raises none of the others.

Round 2 raises three new findings.
The gating one is the attach-time window selection that rides `AttachArgv`'s layout chain, which the new window layout suppresses in the common session where Selvage is alone in the shell window; it also exposes D10's attach pre-flight to the same early return.
The two LOW consistency findings are D10's Warn on a failure that recurs at each open and attach, and D5's newest-agent premise stated without D6's psmux caveat.
