---
round: 1
ledger:
  - key: darn-write-append-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: darn-alignment-judged-against-writer-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: unrecorded-correction-reverted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: commit-race-false-premise
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: constraints-carry-bookkeeping
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d10-no-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
Two BLOCKING design findings concern the darn flow: the append edge during Darn-Write has no mechanical bound, and the alignment approve measures against the writer's own record.
A MEDIUM design finding covers a parent correction the writer followed but nobody recorded, which review would revert.
Three LOW findings are local corrections: a false commit-race premise, Constraints holding bookkeeping, and a missing D10 criterion.
