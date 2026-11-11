# Round 1 fixer-report

## Changes

- F1: fixed as proposed (`93c31070`).
  `internal/adoptcli/check_test.go` has a new `TestRunCheckMarks` row, `FailedAfterAGreenTierWithNoFailedTierNamesBeamStatusAlone`.
  A gate ends tier0 green and exits 1, so the record is failed with a tier and no `FailedTier`.
  The row asserts the final mark is failed with `adopt check of alpha failed; lym beam status shows why`.
- F2: fixed with my own change, in part (`bae02ded`, then a follow-up commit).
  - Made: `TestRunCLI_Down_IsNotRefusedInBolt` in `internal/reedcli/cli_test.go` runs `down` in a Bolt checkout and asserts the refusal text is absent.
    A shared `fabricRefusalText` const now serves it and the refusal test, and the file header names the new test.
  - Not made: the proposed rows running `attach` and `up` in an ordinary task worktree, which are under Deferred.

- Gate repair: the `cli_test.go` header comment I had rewritten wrapped mid-sentence.
  It is now one sentence per line, with no change to its content.

## Disputed

None.

## Deferred

- F2, the `attach` and `up` accepted-path rows in a task worktree.
  Both verbs go on to start a real tmux server once the guard passes.
  The tier0 gate failed with `tmuxkit: a tmux server on key ... outlived the run`.
  Moved to tier2, the run failed with `tmuxkit: ... unregistered key ... outlived the run`.
  Registering the key needs `tmuxkit.KillOnCleanup`, whose helpers live in the `tmux || llm` tagged smoke files.
  The guard is inline in the `PersistentPreRunE` at `internal/reedcli/cli.go:166`, so a hermetic test needs it extracted, outside a test-only fix.
  `up` acceptance in a prime is covered live by `mustRunReed(t, prime, "up")` in `internal/reedcli/smoke_debuglog_test.go`; `attach` acceptance has no test.

## Rulings

None.

## Self-check

All four forks returned a report.

- fix-code-correctness, LOW: the `attach` and `up` rows are absent. Recorded under Deferred.
- fix-consistency, LOW: `fabricRefusalText` duplicates the literal in the refusal test. Fixed: the refusal test now uses the const.
- fix-consistency, LOW: the file header did not name the `down` test. Fixed.
- fix-consistency, LOW: the report must record the missing `attach` and `up` rows. Recorded under Deferred.
- fix-proposal, LOW: the same missing rows. Recorded under Deferred; its suggested direct guard test needs an extraction out of scope.
- fix-proposal, LOW: the duplicated literal and the header. Fixed, as above.
- fix-affected-set, LOW: the same missing rows, `attach` uncovered. Recorded under Deferred.
- fix-affected-set, F1: no findings.
