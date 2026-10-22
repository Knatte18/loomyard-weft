---
round: 3
ledger:
  - key: finalize-reentry-owner-check
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: adopt-back-replays-old-owner-commits
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-adopter-first-repair-one-sided
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: owner-pattern-text-wider-than-design
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
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

Round 3 raises three new keys and reopens one.
`finalize-reentry-owner-check` (BLOCKING): D7's owner check has no exemption for a Finalize re-entered after its parent-side merge, against the declared `PATTERN-owner-machine` text and D6's carve-out.
`adopt-back-replays-old-owner-commits` (third round): the live-pair stop in D5 step 1 assumes `LivePair` reads an owner-paused pair live, but the stock seam reads any pair a live pause entry covers as not live.
`d6-adopter-first-repair-one-sided`: D6's repair resets the code branch only, and its halt note names the reset unconditionally.
`owner-pattern-text-wider-than-design`: the declared PATTERN text says every owner verdict is judged inside `ReconcileOrigin`, while D3 and D5 read the pulled Bolt, the local Bolt and post-push re-reads.
The four round-2 keys other than `adopt-back-replays-old-owner-commits` do not appear in round 3 and are resolved.
