---
round: 4
ledger:
  - key: d11-audit-table-incomplete
    rounds: [1, 2, 3]
    status: resolved
  - key: recovery-branches-unreached-by-tests
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: repair-session-name-keep-verdict-false
    rounds: [3]
    status: resolved
  - key: resize-pins-keep-ignores-fallback-box
    rounds: [3]
    status: resolved
  - key: decision-read-guard-rows-unreachable
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

The D11 table sweep, which recurred for three rounds, drew no finding in round 4, and the two round-3 keys are not raised again; all three are resolved.

`recovery-branches-unreached-by-tests` recurs a fourth time, now as F2: the extended strand-read test probes once and its revive stub fails on any revive, so recovery is never reached.
The fix names the probe cadence, wait length and a non-dead post-heal answer, so it is mechanical.

`decision-read-guard-rows-unreachable` is new and BLOCKING (F1).
Card 9's three `ReadDecision` guard rows cannot reach their sites, because the awaiting arm's own decision read hard-ends the Call on the same check.
The handed-lock site never repeats, so its D11 verdict changes from guard to keep, and the other two rows need pause coverage.
That changes a verdict and a test design, so it needs a fresh review.
