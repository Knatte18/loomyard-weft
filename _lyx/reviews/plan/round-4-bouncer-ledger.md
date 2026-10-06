---
round: 4
ledger:
  - key: card3-prunes-testtiming
    rounds: [1]
    status: resolved
  - key: skill-loading-carveout-uncarried
    rounds: [1]
    status: resolved
  - key: card18-testtiming-in-scope
    rounds: [1]
    status: resolved
  - key: card17-catchall-packages
    rounds: [1]
    status: resolved
  - key: card5-undefined-scenario-merge-ref
    rounds: [1]
    status: resolved
  - key: decision13-bounds-uncarried
    rounds: [1]
    status: resolved
  - key: card18-link-integrity-missing
    rounds: [1]
    status: resolved
  - key: keep-reasons-unbounded
    rounds: [2]
    status: resolved
  - key: card18-leftover-grouping
    rounds: [2]
    status: resolved
  - key: stale-card-number-in-procedure
    rounds: [3]
    status: resolved
  - key: skill-loading-reason-outside-closed-set
    rounds: [3]
    status: resolved
  - key: loop-exit-ignores-reason-class
    rounds: [3]
    status: resolved
  - key: card16-too-many-concerns
    rounds: [3]
    status: resolved
  - key: card19-leaf-label-mismatch
    rounds: [3]
    status: resolved
  - key: cards14-16-mixed-concerns
    rounds: [4]
    status: open
    class: design
    severity: LOW
  - key: card1-failed-verdicts-no-handoff
    rounds: [4]
    status: open
    class: consistency
    severity: LOW
  - key: card13-label-omits-statuscommit
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
  - key: card21-wrong-decision-cite
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 raised nine findings, all resolved by round 3.
Round 3's five findings, including the BLOCKING stale card number and the three MEDIUM ones, were not raised again in round 4, so they are resolved.

Round 4 raises four new findings, none above LOW.
`cards14-16-mixed-concerns` (F1, design, LOW): cards 14 and 16 each pair a slow fixture-merge package with unrelated unit packages.
`card1-failed-verdicts-no-handoff` (F2, LOW): card 1's failed-entry verdicts have no consumer, since each package card reruns its own.
`card13-label-omits-statuscommit` (F3, NIT) and `card21-wrong-decision-cite` (F4, NIT) are label and citation slips.
