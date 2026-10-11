# Plan-Review round 1 — fixer report

Every finding's premise held against the code and the decision record; all thirteen are fixed.
`lyx loom validate-plan` passes on the fixed plan, with only the informational `create-new-unit` findings for the new test files.

## Changes

- **F1 (BLOCKING)**, fixed as proposed.
  Card 9 now edits `internal/battenrecipe#TestNameRunShed_ValueIsPinnedAgainstASymmetryRename` (its four-name row list at `recipe_test.go:147`), `internal/shedrecipe#TestNames` (its `ShipsExpectedEntries` list) and `internal/shedrecipe#minimalEntries` (whose absence fails `TestEveryEntry_RejectsUnknownConfigKey` at `conformance_test.go:87`).
  It also edits the integration members `internal/battencli#fourRowRunForRecipe` (steps `battenrecipe.New`'s embedded recipe), `internal/battencli#stepAwaitingApprovalResumeDoneTeardown_ArchivesTheRunRecords` and `internal/battencli#shortPollBattenRecipe`, all as member glyphs.
  Its Intent says how each changes: Land in the name lists, a Land row in `minimalEntries`, and the integration steps passing Land over the hubforge hub before Worktree-Teardown.
- **F2 (MEDIUM)**, fixed by making the way forward re-run Land.
  Confirmed in `internal/shedengine/run.go`: a Stuck with no `on_stuck` persists blocked with `consumePause` false (line 332), and only step 3 (line 215) or `Goto` (`goto.go:95`) clears `pause_requested`.
  The pause Stuck now names `lyx batten goto <slug> --to Land` then `lyx batten run <slug>`, since the goto clears the flag and leaves the run paused at Land.
  This is stated in Shared Decision `land-row-pause`, in card 8's Intent and its `doc.go` paragraph, and in card 9's `preGoto` text: batten already at Land is the term that admits the goto.
  Card 9 creates `TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun`, which drives pause, goto and the next step through `battenrecipe.New`'s shed and asserts Land is called again.
  Card 9 now raises `internal/battenrecipe`'s budget row.
- **F3 (MEDIUM)**, fixed as proposed.
  Card 8's failing-steps table gains rows for read errors from `ParentBranch`, `PrimeBranch`, `PrimeDirty` and `Landing`, an empty `ParentBranch` answer, a `LandLock.Acquire` error, and a ready-snapshot marker whose scratch path is a directory.
  The empty-parent refusal moved from card 9's wiring into card 8's prime check.
  That puts it in the producer, where a fake seam reaches it, and card 9's `ParentBranch` now only errors on an absent origin record.
  Card 10's tables gain a resume refused with `ErrChildDriverBusy`, which retries and sends the driver-busy notice once, plus `PublishFailure` and `ChildHead` read-error rows that are not retried and get the blocked notice.
- **F4 (MEDIUM)**, fixed as proposed.
  Card 11 rewords `## The run loop` item 2 to name batten's own retry of the first Publish verify failure as the one exception.
  It also adds a `## The run loop` claim and keeps the existing claim on `lyx loom resume` in the task worktree.
- **F5 (MEDIUM)**, fixed as proposed: card 11 targets `contracts/stencils#wordingClaims` (`claims_test.go:107`) and its Intent names that table's `orch-template-role.md` row.
- **F6 (MEDIUM)**, fixed as proposed.
  Card 2 cuts `TestPushBranchWithRetry_RetriesOnlyTransientRefusalsUpToTheBound` to the wrapper's own behavior.
  It keeps a success row pinning `git push -u origin <branch>` in the given dir, and one persistent-refusal row asserting that the seam's `sleep` is used and the error wraps `gitrepo.ErrPushRetriesExhausted`.
  The refusal-then-success, hook-declined and non-fast-forward rows are dropped, and it gains no `cannot lock ref` row.
- **F7 (LOW)**, fixed differently.
  The proposal was to extend `TestLoadConfig_PublishVerify` in place.
  With `deploy` rows its name would no longer describe its rows, so card 5 deletes it and creates `TestLoadConfig_OptionalStringKeys`, one table over `publish_verify` and `deploy`, each set and absent.
  The `TestLoadConfig_Deploy` create and the budget raise are dropped as proposed, and `internal/landingshed`'s count is unchanged.
- **F8 (LOW)**, fixed differently, for the same naming reason.
  Instead of extending `TestWire_PauseRequestedReadsBattensOwnStatus` in place, card 7 deletes it and creates `TestWire_BattenStatusSeamsReadBattensOwnStatus`.
  The new test is one table over the seams reading batten's own status: its existing `InnerRun.PauseRequested` rows plus `Teardown.Notices.BattenHistoryLen` rows.
  Card 9 adds `Land.PauseRequested` rows to it through its `plan:` handle.
  `TestWire_RowNoticesReadBattensOwnHistory` and the `internal/battencli` budget raise are dropped from card 7.
- **F9 (LOW)**, fixed as proposed (second option): `conditionText` leaves card 10's targets.
  Instead, `noticeCondition` sets the finding's `text`, which `conditionText` already renders after the halt text (`notice.go:542`).
- **F10 (LOW)**, fixed as proposed (the escape hatch is kept).
  Card 9's Intent states that the existing goto onto Worktree-Teardown with a done child now skips Land's pull, deploy, prune and ready notice.
  It keeps that goto as the operator's way past a Land step settled by hand, and states what bounds it.
- **F11 (LOW)**, fixed as proposed: card 9 edits `internal/battencli#haltedOverDoneChild` and `internal/battencli#TestHaltedOverDoneChild`.
  The flag now says `lyx batten run <slug>` resumes at the halted row and lands through Land before Worktree-Teardown.
- **F12 (LOW)**, fixed as proposed.
  Card 11 edits `internal/orchcli/doc.go` so step 5 no longer has the orch pull, deploy, reconcile or remove the pair.
  Step 3 also drops the `lyx loom start` after `lyx loom approve`, which D7 retires in the same text.
- **F13 (LOW)**, fixed as proposed: card 1 declares `plan:internal/gitrepo#retryablePushRefusal`, unexported, and its in-package test reaches it.

## Disputed

None.

## Deferred

None.
