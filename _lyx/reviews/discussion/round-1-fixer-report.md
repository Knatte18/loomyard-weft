# Fixer report: discussion round 1

Every finding was checked against the code and holds; all ten are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.

## Fixed

- **F1** (BLOCKING): `verifyGate.recordFailure` in `internal/landingshed/verifygate.go` writes no record for a timeout or a shell that could not start.
  D1 now names the goto only for a verify that ran and exited non-zero, and states that the timed-out and unstartable reasons stay as today with no goto, and why.
  The Goal bullet, the Scope In line and the first Acceptance criterion match.
- **F2** (MEDIUM): `start` calls `refuseOverUnfinishedMerge` whenever it must spawn, whatever the run's state.
  D3's exception now covers any run whose current producer is Publish or Finalize, halted there or `running` with a dead driver, and its Rationale names `start` as the respawn path.
  Scope, the Auto-mode assumption and the Acceptance criterion match.
- **F3** (MEDIUM): `reedengine.Geometry.NameSlug` is empty in a standalone run.
  D6 now also refuses any non-empty strand name in a standalone run, explains why in the Rationale, includes the standalone state in the Bound, and gives the plan stencil text a no-parent branch: stop and report the addition.
  The Auto-mode assumption and the Acceptance criterion match.
- **F4** (MEDIUM): D6's way forward now names the run's recorded parent, else the operator, and never spells a shortname as a literal; the stencil text says "your parent".
  The Acceptance criterion requires that the stencil names no shortname.
- **F5** (LOW): `isTerminalOutcome` in `internal/shuttleengine/attach.go` already lists `runOutcomeStopped`.
  D5 is now a test-only Decision adding `stopped` rows to the existing tables, with the conditional dropped; the Auto-mode assumption matches.
- **F6** (LOW): D1's Bound now says Publish verifies again, skipping only a tree its verified-tree record names under the same command, and removes the failure record only after both verifies pass or skip.
- **F7** (LOW): D2's Bound now states that the abort discards any parked parent merge-in into the task pair, whoever started it, including a hand-run `lyx fabric merge-in`.
  The own-leftover test also checks the record's verb is `merge-in`, so a hand-run `lyx fabric merge <parent>` in the pair, whose `mergeState.Verb` is `merge` (`internal/fabricengine/merge.go`), is never aborted.
  D3's Bound now points at D2's instead of naming the orch case alone, and the D2 and D3 Acceptance criteria and the `MidMerge` plan-writer note name the verb.
- **F8** (LOW): the Acceptance criterion that both Webster-Review rows render the Publish failure section is removed.
- **F9** (NIT): the `webster-replan-state` and `driver-context` Constraints are removed.
- **F10** (NIT): the Open risks entries for D1, D2, D3 and D6 are removed; the D4 entry stays.

## Disputed

None.

## Deferred

None.
