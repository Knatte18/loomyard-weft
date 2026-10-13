---
round: 1
ledger:
  - key: d7-retry-on-unreached-path
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d1-tool-event-proof-unproduced
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-cites-d3-for-launch-line
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-verified-send-bypass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-root-resolution-partial-write-bounds
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: scope-item-4-no-acceptance-criterion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, two of them gating.
The D7 retry (F1, BLOCKING) targets the contained removal, but the sighted failure is `git worktree remove` itself, which unregisters the worktree and leaves the directory, so the retry never runs.
D1's readiness proof (F2, MEDIUM) accepts a tool event the events file never records.
The remaining four are LOW: a wrong cross-reference (F3), an unbounded verified-send bypass in D2 (F4), D6 root resolution, partial write and missing acceptance bounds (F5), and a missing acceptance criterion for scope item 4 (F6).
