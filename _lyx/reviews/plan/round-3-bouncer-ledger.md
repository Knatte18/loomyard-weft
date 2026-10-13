---
round: 3
ledger:
  - key: d11-audit-table-incomplete
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: recovery-branches-unreached-by-tests
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: repair-session-name-keep-verdict-false
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: resize-pins-keep-ignores-fallback-box
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

`d11-audit-table-incomplete` has now recurred three rounds running, at BLOCKING since round 2.
Each fix added the rows the review named, but the sweep still stops one call level down and at the callback's package edge.
Round 3 names reed's state-load Warns, shuttle's record readers reached through loop callbacks, battencli's session-state reader and parentlock's release Warn.
The remedy is a full per-iteration call-graph re-sweep, not more rows, so it needs a fresh review.

`recovery-branches-unreached-by-tests` recurs at MEDIUM.
Round 3 finds it on card 9's `reviewWaitNote`, whose extended test row fails on every check and never reaches the recovery line.
The remedy edits a test row already named, so it is mechanical.

`repair-session-name-keep-verdict-false` is new and BLOCKING.
The keep verdict for `repairSessionName`'s capture and type failures does not hold, since both Warns repeat on every wake while the name drifts.
The finding spells out the verdict change, the episode guard and the test rows.

`resize-pins-keep-ignores-fallback-box` is new and LOW.
The keep rationale for `runResizePinsLocked` ignores the fallback-box case, where the pins run on every tick.
