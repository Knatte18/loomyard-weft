---
round: 1
ledger:
  - key: d7-halt-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: adopt-back-replays-old-owner-commits
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-lock-outlives-handover
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slug-pause-exempt-for-lock-holder
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: shed-verb-check-vs-batten-child-spawns
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: start-check-after-seed-claim-order
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopt-all-vs-done-and-unstarted
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: constraints-owner-reads-understated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: shed-resume-unowned-run-held
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: owner-pause-marking-identity
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: owner-write-lost-replay-unchecked
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-same-machine-resume-window
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d7-deliberation-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first judged round, so there is no earlier ledger and no key recurs.
Keys map to F1 through F14 in order.
The blocking gap is D7's halt message, which names ways forward that do not land the run.
Four more design gaps concern the handover mechanics: adopting back, the parent lock a halted run keeps, the slug pause exemption for a lock holder, and the shed-verb check against batten's own child spawns.
A fifth design gap is the start check's order against the seed and claim.
The rest are consistency and wording corrections whose fix text is already given.
