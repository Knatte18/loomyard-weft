---
round: 3
focus:
  - "Carry-over consumers: for each of the three review segments, does the Discussion's decision record state which downstream consumer (Plan-Write, the Plan-Review rubric, Describe, the rework stencil) is told to read a carry-over entry in `## Open risks` and what it does with it, and does Scope list the stencil changes that wiring needs? Settle whether the Open risks bullet and the Goal claim a guard that exists."
  - "D1 degraded read: in `internal/shedadapters/bouncerfacts.go`, `ledgerHistories` lets an unreadable or unparseable ledger contribute nothing, and `judgeCall` in `internal/shedadapters/bouncer.go` falls back to `(none)` for an unparseable previous ledger. Does the decision record state what D1's earlier-open set yields in that case, and whether the Bound and acceptance criteria cover it?"
  - "D2 entry shape and bound: does the record fix how a carry-over entry is marked and where it ends inside `## Open risks`, which also holds hand-written bullets, and name the told value that carries the segment identity? Does the stated bound on the Go write hold, given `discussionparser.Validate` in `internal/discussionparser/validate.go` checks heading presence only and takes a support log path, and given the judge-written ledger key is an unconstrained scalar?"
  - "D2 lifetime and path form: does the record say what happens to an entry when a later generation settles without a ledger or never reaches `Done`, and which path form and rounds source the entry uses in a record that is committed and read in other worktrees?"
---

Each entry restates the sites it depends on; check each against the current decision record text and report only what the record still leaves unsettled.
