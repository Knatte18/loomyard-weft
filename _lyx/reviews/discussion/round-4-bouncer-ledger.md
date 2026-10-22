---
round: 4
ledger:
  - key: owner-pattern-text-wider-than-design
    rounds: [3, 4]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: adopt-back-replays-old-owner-commits
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-adopter-first-repair-one-sided
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-finalize-running-detector
    rounds: [4]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-reentry-owner-check
    rounds: [3, 4]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d6-reset-drops-other-local-commits
    rounds: [4]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: slug-pause-exempt-for-lock-holder
    rounds: [1, 2]
    status: resolved
  - key: shed-verb-check-vs-batten-child-spawns
    rounds: [1, 2]
    status: resolved
  - key: d7-halt-way-forward
    rounds: [1, 2]
    status: resolved
  - key: offline-fallback-unbounded
    rounds: [2]
    status: resolved
  - key: parent-lock-outlives-handover
    rounds: [1]
    status: resolved
  - key: start-check-after-seed-claim-order
    rounds: [1]
    status: resolved
  - key: adopt-all-vs-done-and-unstarted
    rounds: [1]
    status: resolved
  - key: constraints-owner-reads-understated
    rounds: [1]
    status: resolved
  - key: shed-resume-unowned-run-held
    rounds: [1]
    status: resolved
  - key: owner-pause-marking-identity
    rounds: [1]
    status: resolved
  - key: owner-write-lost-replay-unchecked
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: d8-same-machine-resume-window
    rounds: [1]
    status: resolved
  - key: d7-deliberation-narrative
    rounds: [1]
    status: resolved
---

Round 4 raises six findings, one BLOCKING.
`owner-pattern-text-wider-than-design` (F1, BLOCKING, consistency): the declared `PATTERN-owner-machine` text says only the owner acts, while D2 and D3 let any machine act on an unowned entry.
`adopt-back-replays-old-owner-commits` (F2, fourth round): the report never carries a live pair for a branch the reconcile does not move, so adopt must ask `LivePair` itself.
`d6-adopter-first-repair-one-sided` (F3, second round): `finalizeEntered` keeps an ended run its own Finalize reached out of the ended set, so D6's teardown claim and records-commit claim do not hold.
`d6-finalize-running-detector` (F4, new): D6 names no signal for "Finalize is running" beyond `parentHoldsTaskWork`.
`finalize-reentry-owner-check` (F5, reopened at LOW): the skip covers only the shed driver's in-process re-step, and the unreachable-origin halt is not stated as skipped.
`d6-reset-drops-other-local-commits` (F6, new): the halt note's reset drops every local parent commit origin lacks.
Round 3's `finalize-reentry-owner-check` BLOCKING is down to LOW.
All round 1 and round 2 keys other than `adopt-back-replays-old-owner-commits` stay resolved.
