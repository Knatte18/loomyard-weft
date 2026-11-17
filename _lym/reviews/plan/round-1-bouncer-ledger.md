---
round: 1
ledger:
  - key: test-only-code-first-caller-later-card
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: daemon-refusal-not-fakeable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bridge-peer-command-failure-outcome
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: deliver-verb-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card10-extends-rewrites-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: own-name-check-case
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: duplicate-bridge-receive-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: release-gone-holder-redundant
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: trustfield-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings, none carried from an earlier round.
The gating set is the BLOCKING reachability gap (cards 1, 4, 6, 7 and 8 create production functions whose first caller is a later card)
and three MEDIUM design gaps: an unfakeable daemon refusal, an undefined bridge-channel peer-command failure outcome, and an unbounded deliver verb.
The remaining five are local consistency corrections.
