# Fixer report — discussion round 1

Every finding is fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1 (BLOCKING):** D3 now requires that a gated run's turn end with background work outstanding (`EventWaiting`) is no arrival, even when every output file exists, and states the bound: live session only, with deadline and liveness exits still classifying Done from the files under `master_timeout_min`.
  Scope's webster-gate bullet names the shuttle change.
  The false Constraints entry is removed, and an acceptance test covers the fixer-spawning turn end.
- **F2 (MEDIUM):** D5's commit check accepts a clean parent merge on the first-parent chain, as record-batch does, and an acceptance test covers both merge cases.
- **F3 (MEDIUM):** D5 no longer claims the commit check guards plan or `_lyx` paths.
  The run-exit fork audit (`CheckFork`) is named as the guard on fixer writes there, and is added to the bound list.
  The impossible acceptance test is replaced by one on the run-exit audit.
- **F4 (MEDIUM):** D7's bound states that only the shared function writes the record, that no agent prompt names its path, and that an agent writing it is outside a steering-guard bound.
- **F5 (MEDIUM):** D8 gains a Bound line: no record on failure, so Publish's D7 verify runs and is Stuck before any push.
  The Open risks entry keeps only the remaining cost.
- **F6 (LOW):** D1 gains a Bound line (the fork runs the gate, Go never does, the webster gate catches a skip), and the duplicate Open risks entry is removed.
  The rationale drops "mechanical", which the spec uses for no-author-enumeration, not Go enforcement.
- **F7 (LOW):** D1 states that its integration-tagged run moves part of Tier 2 per card and that the spec's Tier 2 text changes.
  The Notes entry on `loom-plan-spec.md` says the same.
- **F8 (LOW):** D3 step 1 states that an absent `outcome.yaml` cannot reach the gate and errors at the post-gate parse if it did, and that a non-`done` outcome never reaches Publish.
- **F9 (LOW):** Acceptance criteria gain D2's template test, D6's blocking `recover-batch` and `poll_wait_s` loading, and D4's `verify-gate.yaml` report.
- **F10 (NIT):** The card 07/card 09 sentence is removed from D4; the support log's Interview item 3 keeps it.

## Deferred

None.
