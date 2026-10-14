---
round: 2
ledger:
  - key: darn-write-append-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-alignment-judged-against-writer-record
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: unrecorded-correction-reverted
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: commit-race-false-premise
    rounds: [1]
    status: resolved
  - key: constraints-carry-bookkeeping
    rounds: [1]
    status: resolved
  - key: d10-no-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: carried-over-correction-no-pr-align-rule
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1's two BLOCKING design findings were reduced but their themes persist.
The append-edge finding returns as F3: the dropped-entry checks measure against a committed copy the writer can move.
The alignment-measuring-stick finding returns as F4: D10 moved the measuring stick but left the scope-deviation definition tied to the writer's own decisions.
The unrecorded parent correction returns as F1 at BLOCKING: D7's "finding left open" has no mechanism in the Darn-Review perch.
F2 is new: PR-Align has no rule for a carried-over unrecorded correction, so it rejects and bounces to Darn-Write.
The three round 1 LOW findings are not raised again and are resolved.
