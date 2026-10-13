---
round: 2
ledger:
  - key: d7-retry-on-unreached-path
    rounds: [1]
    status: resolved
  - key: d1-tool-event-proof-unproduced
    rounds: [1]
    status: resolved
  - key: d1-cites-d3-for-launch-line
    rounds: [1]
    status: resolved
  - key: d2-verified-send-bypass-unbounded
    rounds: [1]
    status: resolved
  - key: d6-root-resolution-partial-write-bounds
    rounds: [1]
    status: resolved
  - key: scope-item-4-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: d7-windows-sharing-violation
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-recognition-reads-stderr
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-leftover-removal-no-mutation
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1's six findings do not reappear in round 2, so all are resolved.
Round 2 raised three new findings, all on D7.
F1 (MEDIUM, design): on Windows a held reed lock file fails the leftover removal with a sharing violation, so the retry misses its target window.
F2 (LOW, consistency): the not-empty recognition sentence implies reading git's stderr.
F3 (LOW, design): the leftover removal records no mutation, so Remove's envelope omits the deleted directory.
