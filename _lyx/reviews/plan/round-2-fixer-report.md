# Plan-Review round 2 — fixer report

All four findings validated against the code and fixed; `lyx loom validate-plan` reports `ok: true` with informational findings only.

## Fixed

- **F1** (`03-teardown-settles-board-claim.md`): premise confirmed — `state.ReadJSONStrict` takes `lock.AcquireReadLock` first, and gofrs/flock's open fails when the task worktree is gone.
  The Intent now has `RemovePair` ask the existing `gone` substrate first: a gone task worktree is not landed, with no read and no error, so the settle falls through to the batten-state rule.
  `TestTeardown_CallSequence` gains a gone-pair row over a run-form entry with no batten seed, asserting the clear and no `landed` read.
- **F2** (`03-teardown-settles-board-claim.md`): `internal/pairteardown/teardown_integration_test.go` joins `Edit:`, and `TestRun_TeardownScenario` gains steps driving the real claim substrates over hubforge pairs: no batten seed clears, a child `Finalize` Done marks done, a `running` prime batten status leaves the entry unchanged.
  The file is `tmux`-tier, compiled by the plan's `go vet -tags tmux` and run at Publish, so `## verify:` is unchanged; no new top-level test, so no test-budget change.
- **F3** (`13-batten-admits-darn.md`): the bound sentence now matches card 12: batten admits `darn` only from the operator-set `recipe`, the one other writer of a darn seed is a hand `lyx shed seed`, and `lyx loom start` never rewrites a recorded recipe.
- **F4** (`06-verify-gate-told-command.md`): the whole-diff form states that a `mergeBase` read error is returned as the gate's error with its message kept, never a skipped lint, and the gate scenarios gain a step with a failing `mergeBase` reader asserting the error and that nothing runs.

## Disputed

None.

## Deferred

None.
