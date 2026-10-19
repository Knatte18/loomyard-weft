---
round: 2
ledger:
  - key: gitexec-probe-unlogged-stale-exemption
    rounds: [1]
    status: resolved
  - key: d2-skip-drops-empty-from-start-commit
    rounds: [1]
    status: resolved
  - key: d3-tmux-34-session-order
    rounds: [1]
    status: resolved
  - key: d6-marker-does-not-identify-probe
    rounds: [1]
    status: resolved
  - key: d4-quarry-gitsrc-bypasses-floor-check
    rounds: [1]
    status: resolved
  - key: d4-process-cache-vs-per-test-hooks
    rounds: [1]
    status: resolved
  - key: no-acceptance-criterion-tmux-floor
    rounds: [1]
    status: resolved
  - key: d7-criterion-needs-undeclared-seam
    rounds: [1]
    status: resolved
  - key: d3-attach-repin-unmarked-prime
    rounds: [1]
    status: resolved
  - key: rejected-alternative-and-history-prose
    rounds: [1]
    status: resolved
  - key: apple-git-in-admitted-set
    rounds: [1]
    status: resolved
  - key: d3-boot-cannot-mark-existing-prime
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-probe-count-leaves-tree-comparison-channel
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's eleven findings are all resolved: round 2 raised none of them, and its focus verdicts found D4, D2's empty-from-start handling, D6's marker and D7's seam sound.
Round 2 raised two new findings.
F1 is D3's false claim that a boot marks a prime session an older binary created; only the attach pre-flight can, so D3 needs the bound that such a session draws in the second loop until an attach.
F2 is D2's probe count leaving the tree comparison's channel unstated.
