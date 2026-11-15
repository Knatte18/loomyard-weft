# Fixer report — round 1

## Changes

- **F1 (BLOCKING), fixed as proposed, plus a recheck of origin.**
  `battencli.ClearDoneRun` now builds the hub topology and passes `clearDoneRun` a branch-discard function.
  `discardTaskBranchIfPresent` archives and deletes the run's task branch, locally and on origin, through `Topology.DiscardTaskBranch`.
  It skips the call when neither the local branch nor origin holds one, as the cancel path does.
  The discard runs after the run is judged and before any record is removed, so a failure returns the error with the records in place and the dobby's next tick retries.
  `ClearMotifRun` passes nil and keeps its behaviour.
  `DiscardTaskBranch` takes the teardown lock and reaches origin through `gitrepo`, so the remote-refs lock is held where the boundary pattern requires; no raw git was added.
  A test row (`AnyRecipe_TaskBranchRemovalFailureReturnsTheErrorAndKeepsTheRun`) and a `wantDiscards` assertion in `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` pin the removal, its order before the records, and the kept records on failure.
  The self-check lens `fix-proposal` showed that the origin half can be handed to another process and fail without an error to the caller.
  `discardTaskBranchIfPresent` therefore reads origin again after a nil discard and returns an error while the branch is still there.
  `doneSlugRefusal` stays as it was: the branch is removed, so the review's condition for editing it does not hold, and its text stays true until the dobby clears the run.
- **F2 (MEDIUM), fixed as proposed.**
  `internal/beamcli/doc.go`, `docs/overview.md` (the `beam create` paragraph and the `lym batten cleanup` paragraph) now say the parent's dobby removes a done run once its landing is counted and its notice decided, and that the refusal applies until then.
  The self-check found the same claim in `internal/beamcli/cli.go` (the `lym beam create` help) and `internal/beamcli/create.go`; both are updated.
  The `internal/dobbyengine` docs, the `Deps.ClearDoneRun` comment and the `donerun.go` header and function comment now name the task branch.
- **F3 (LOW), fixed as proposed.**
  `TestClearDoneRuns_WaitsForTheLandingCountOfARunWhileATidyIsInFlight` in `internal/dobbyengine/donerun_test.go` drives `countLandings` then `clearDoneRuns` over the tidy harness's fake `Deps.Landings`.
  A done run that landed is not cleared while a tidy is in flight, and is cleared once the tidy settles.
  It fails if `d.landed = landed` is removed from `countLandings`.
  The `d.clearDoneRuns(now)` call in `tick` is still unpinned: a tick test needs the whole daemon fixture, and the finding allowed driving `countLandings` and `clearDoneRuns` instead.
- **F4 (LOW), fixed as proposed.**
  `clearDoneRuns` now drops, in its one `UpdateState` call, every counted landing whose slug is neither landed nor listed, besides the cleared slugs.
  A missed deletion is repaired at the next tick, and the warning for a failed update stays.
  `TestClearDoneRuns` seeds a `gone` entry in every row and asserts it is dropped, and keeps the `other` entry by listing that run.
- **F5 (LOW), fixed as proposed.**
  The `AnyRecipe_DoneRunWithNoChildRecipeRecordIsRemoved` and `AnyRecipe_DoneDarnRunIsRemovedCommittedAndPushed` rows now carry `wantMessage`, which the test compares with the records commit message.

Commits, none pushed: `1850b2e7` (F1 and F5), `57c9605b` (F2 docs), `967d21bb` (F3, F4 and the dobby docs), and one commit for the self-check changes.
F1 and F5 share a commit because both change the same table-driven test.
`lym gate test` over `./internal/battencli`, `./internal/dobbyengine` and `./internal/beamcli` passes.

## Gate repair

The line-length gate flagged six comment lines this round added in `cleanup.go`, `donerun.go` and `loop.go`.
Each was split at a sentence boundary into shorter lines, with no change of meaning.
A second gate pass flagged the wrapped sentence of the `ClearDoneRun` comment that began "When a records removal fails".
It is now two sentences, one per line.
Two further commits carry the repairs.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: `internal/beamcli/create.go` and `internal/beamcli/cli.go` still say a done run is refused — fixed as proposed.
- fix-consistency, LOW: `donerun.go` header and `clearDoneRuns` comment describe records only — fixed as proposed.
- fix-consistency, LOW: `cleanup.go` file header omits the task branch — fixed as proposed.
- fix-proposal, MEDIUM: the origin half of the discard can fail after `DiscardTaskBranch` returns nil — fixed as proposed, by the origin recheck above.
- fix-affected-set, MEDIUM: the `lym beam create` help in `cli.go` — duplicate of the consistency finding, fixed.
- fix-affected-set, LOW: `doneSlugRefusal` does not mention the dobby's clear — fixed with my own change: none made. The lens itself calls the text accurate for the window before the clear, and the review asked for an edit only if the branch stayed.
- fix-affected-set, LOW: `internal/dobbyengine/doc.go:485` — not stale by the lens's own reading; no change.

## Disputed

None.

## Deferred

None.

## Rulings

None.
