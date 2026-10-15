---
verdict: APPROVED
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, 14 commits (cards 1 to 14); `_lyx/loom/rework` does not exist, and `product.parent` in `_lyx/shed/run-pane-timing/status.json` is `main`.
No Publish failure is on record.
`lyx gate test` over the touched packages (`shuttleengine`, `shedengine`, `shedadapters`, `loomshed`, `loomrecipe`, `loomcli`, `battencli`, `statuspane`, `shedrecipe`, `burlerengine`, `websterengine`, `contracts/...`, `cmd/lyx`) is green, `cmd/lyx` covering the test budget.

## Focus directive verdicts

- **Gate spans.**
  Spans are measured only in `evaluateGate`, `handleGatedBoundary`, `Wait` and `finalize` in `internal/shuttleengine`, all from `run.clock`.
  `shedengine.GateSpan` is a plain struct of strings, with RFC 3339 times from `time.RFC3339Nano`.
  No two spans of one entry overlap.
  The gate span is suppressed while a held span is open.
  The held span opens at the evaluation's end.
  Rework spans close before the final evaluation's gate span.
  `withGateSpans` stamps copies, so the memoised verdict is never written through.
  Not a defect.
- **D1.**
  Closed spans use `statuspane.FormatSeconds` and any time that includes the open visit uses `FormatMinutes`, both through `checklistTime`.
  The gate lines are always closed and use `FormatSeconds`.
  Not a defect.
- **Entry numbers.**
  `checklist.go` (`renderChecklist`) and `battencli/overview.go` (`overviewEntryNumber`) both read numbers from `loomrecipe.EntryNumbers` alone.
  `loomrecipe.Rows` now serves `loom`, `darn` and `batten`, the three `shedrun.Recipe*` values.
  Not a defect.
- **Carry of gate count and spans.**
  Every site that returns a `GateAttempts` pointer also sets `GateSpans`.
  That covers the multi-LLM chair, burler review, burler fix and webster adapters, and `prRework`'s Done and plan-parse Stuck exits.
  A non-Done session outcome passes the session's pointer through unchanged.
  `HistoryEntry` is built with `GateSpans` at `run.go` and all three sites in `reroute.go`, and `appendOrFold` appends spans on a fold without writing through shared arrays.
  Not a defect.
- **Role prefixes.**
  Roles travel as strings.
  `rolePrefixKey` validates through `agentname.ValidateRole`.
  `seatengine` and `bouncer.role` only concatenate, and `agentname.Format` forms the name through reed.
  The per-segment prefixes of the shipped recipes are pinned by the `RolePrefixes` subtests in `internal/loomrecipe/bouncerrows_test.go`.
  `PR-Review` holds no Bouncer row, so it correctly takes none.
  Not a defect.
- **Per-card mechanical check.**
  No card in the plan carries a `Rename` or `Delete` group that the diff left unchecked.
  Cards 2 and 14 only edit and create.
- **Test economy.**
  The diff adds one top-level test, `TestEntryNumbers`, and the `internal/loomrecipe` budget row moved from 54 to 55 in the same diff.
  Every other new case is a row or an assertion in an existing test.
  Not a defect.
