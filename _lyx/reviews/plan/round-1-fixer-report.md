# Fixer report — plan, round 1

Every finding was checked against the code and the decision record, and each premise held, so all eight are fixed.
`lyx loom validate-plan` passes after the fixes; its one informational finding (`premerge_test.go` as a new unit) predates this round.

## Fixed

- **F1** (conflict stopper misses a numbered strand): confirmed in `internal/reedengine/strand.go`, which forms the role through `agentname.NumberRole`, and `maxConflictAttempts = 2` in `internal/mergeresolve`.
  Card 4 gains `plan:internal/agentname#MatchesRole` (`func MatchesRole(role, base string) bool`: `base` or `base-N`, `N` ≥ 2), so the numbered-role match stays inside `internal/agentname` per `PATTERN-agent-name`; `TestNumberRole` gains its assertions, so no top-level test is added. Because card 4 now has `Edit`, it also gets an `ImpactSummary`.
  The card index line for card 4 names the helper.
  In card 5, `conflictSessionStopper` stops every live, non-retiring strand whose role matches `ConflictRole`, and returns the failing guid beside its error. The `StopConflictSession` seam description now matches.
  The stopper uses `MatchesRole`.
  `TestConflictSessionStopper` gains rows for a live `conflict-2` strand and for two live conflict strands.
- **F2** (two ways forward): confirmed against `verifygate.go`'s failed reason. In card 3, the goto replaces the trailing `fix forward on the task branch, then resume` clause when `recorded` is true.
  `TestPublishVerify_FailureRecord` asserts that the recorded rows do not contain that clause.
- **F3** (the parent-recorded refusal omits the unnamed shell): in card 7, both refusal variants end with `the operator's own unnamed shell may always run it`, and the refusal rows assert it.
- **F4** (the abort's reach is unbounded): card 5's Intent states that the abort discards any parked parent-sourced `merge-in`, including a hand-started one with its partial resolution, followed by D2's bounds.
  Card 6's Intent states that the guard lets through exactly that, followed by D3's bound.
- **F5** (the caller check states no bound): card 7's Intent states D6's bound. The check reads only the exported name and the recorded parent name, so it guards a mistaken call, not a forged or cleared variable.
  An unresolvable parent never passes a named caller.
  No driver exemption is given.
- **F6** (which side the SHAs come from): confirmed against `mergeState`'s doc in `internal/fabricengine/mergestate.go`. Card 1 fills `SourceSHA` and `StartSHA` from `WarpSource` and `WarpStart`.
  The test asserts them against the task worktree's code HEAD before the merge and the parent code branch's SHA.
- **F7** (the new seam is outside the seam test): card 6 lists `internal/loomcli/cli_test.go`. `TestNewLoomCLI_SetsInjectedSeams` asserts that `recordedParentBranch` is non-nil, and its `testtiming:keep` line names it.
- **F8** (the `ProbeGated` sub-case is redundant): confirmed that `Attach` and `ProbeGated` share the one `attach` body. Card 8 drops the sub-case.
  Its Intent now says that `TestEngine_ProbeRound`'s existing gone and done rows are the burler coverage D5 asks for, because the burler probe sees only shuttle's not-found answer.

## Disputed

None.

## Deferred

None.
