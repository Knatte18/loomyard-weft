# Fixer-report, round 2

## Changes

All five findings were fixed as proposed, except where marked.
The five fixes went into one commit, `fa3ac4cd`, not one per finding: F1, F2 and F4 all edit `internal/loomcli/prepush.go`, and an interactive partial add is not available here.
The self-check fixes are a second commit.

- **F1** — `internal/fabricengine/failedpush.go` exports `RepoCode` and `RepoRecords`, and after the self-check `RepoBolt`, each set equal to its `failedPushSide*` constant.
  `codeRepo` is deleted from `internal/loomcli/prepush.go`; `parentOutcome` and `stoppedBranches` use the exported constants.
  `internal/loomcli/prepush_test.go` and `internal/loomcli/commitrecords_test.go` build their outcomes and records with them.
- **F2** — `shedrun.ParentNoticeFile` sits beside `PullRequestRecordLock` in `internal/shedrun/paths.go`, with a row in `TestPathConstructors` and a line in `doc.go`.
  `parentNoticeFile` is deleted from `prepush.go`, and `newPrePushCheck`'s `parentNoticed` seam calls the shedrun function.
  I put it in `paths.go`, not `landed.go`, because the landed-commit file's header names that file's one concern.
  This differs from the review's "beside LandedCommitFile" in the file only; the function is a sibling scratch path as proposed.
- **F3** — fixed with my own change.
  The review proposed a hub-fixture test whose origin record names an owner elsewhere; I did not make it.
  Building that fixture needs an owner written to the board and pushed, which is heavy, and the wiring can be pinned without it.
  Instead `loomCommitStatusDeps` and `pullRequestDeps` delegate to `commitStatusDepsGuardedBy` and `pullRequestDepsGuardedBy`, which take the check as a parameter.
  `TestPrePushCheckWiredIntoDeps` passes a stopping check and asserts Commit, Push and PushTask return `errPushStopped` and Spawn returns nil, none reaching git.
  `TestNewPrePushCheckSkip` drives the real `skip` closure over a hub fixture for pushes off, the prime, a pair at Finalize, and a plain pair.
  `stoppingPrePushCheck` is shared with `TestPrePushCheckGuardedSeams`.
- **F4** — `stoppedBranches` skips a records branch in state busy; the row `StoppedSharedBranchBesideOwnBusyRecords` is added to `TestPrePushCheckGuard`.
- **F5** — the `PATTERN-owner-machine.md` bullet names git or pushes off, the prime, Finalize and a pause as the skips, and the failed or unreachable check and an ended entry whose parent holds the work as the cases that go on.

Gate: `lym gate test` tier 0 and tier 1 over `./internal/loomcli` and `./internal/fabricengine` pass, and tier 0 over `./internal/shedrun` passes.

Gate repair: the verify gate failed `TestTierPurity_UntaggedTestsSpawnNothing` and `TestTierTags_OneTierPerTestFile`, because the tier 1 `TestNewPrePushCheckSkip` sat in the tier 0 `prepush_test.go`.
It moved unchanged into `internal/loomcli/prepush_skip_integration_test.go`, a file of its own.

## Self-check

Four forks returned; their reports are in `round-2-fixer-report.lens-<lens>.md`.

- fix-consistency, LOW — `prepush_test.go` `boltMain` kept the literal `"bolt"`: fixed by exporting `RepoBolt` and using it.
- fix-consistency, LOW — PATTERN bullet omitted git off: fixed, "git or pushes off".
- fix-consistency, LOW — `RepoCode` and `failedPushSideCode` are two names for one value: no change; it is what F1's fix proposed, and each constant is the other's definition.
- fix-code-correctness, LOW — PATTERN bullet omitted git off: duplicate of the above, fixed.
- fix-code-correctness, LOW — no row for the producer read of the `skip` closure: fixed with the `PairAtFinalize` row.
- fix-proposal, LOW — `boltMain` literal: duplicate, fixed.
- fix-proposal, LOW — the wiring test calls the `GuardedBy` functions, so a wrong run id inside `loomCommitStatusDeps` or `pullRequestDeps` is not caught: deferred, see below.
- fix-proposal, LOW — other packages' tests keep bare repo literals: no change, outside the review's scope and not made worse by the fix.
- fix-affected-set, LOW — `commitrecords_test.go` literals: fixed.
- fix-affected-set, LOW — other loomcli commits and batten and tender pushes skip the check: no change; the check guards a run's commit for a push and loom's own pushes, as the PATTERN bullet states.

## Disputed

None.

## Deferred

- fix-proposal's run-id case: `loomCommitStatusDeps` and `pullRequestDeps` are one-line delegations to the tested `GuardedBy` functions, and the tier 1 `TestCommitStatusSeam_Real` already runs `loomCommitStatusDeps` over a real pair.
  Pinning the run id through them needs the owner-elsewhere hub fixture that F3's review proposal names and I judged too heavy for this round.
  An operator decision on whether that fixture is worth building would settle it.
