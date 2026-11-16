# Plan round 1 — fixer report

Every finding's premise held against the code and the decision record; all 18 are fixed.
`lym loom validate-plan` reports `ok: true` with informational findings only, after the fixes and again after the self-check.

## Changes

- **F1** (card 13): after the post-batch checks pass on a clean tree, `reenter` runs each card's `card` gate call and the batch's `batch` gate call at HEAD through `RunDeps.Verify` (`verifytree.Verify` in held gate slots with the pause hold), and records the batch done with no spawn only when every gate passes; a failing gate gives the last-rung attempt.
  `TestRun_Reentry` gains the card-gate-fails-at-HEAD row.
- **F2** (card 2): `verify_gate` is `CategoryGuard`; `ladder` and `fixer_timeout_min` stay `CategoryNone`.
  `TestModules_EveryTemplateKeyClassified` moved from Uses to Edit, gaining `wantLocalDenied["weaver"] = [verify_gate]`.
  The Intent states the bound: the local layer cannot change `verify_gate`.
- **F3** (card 1): dropped the `runplan#ConflictRecordRel` Create bullet and its Intent line; `websterengine.ConflictRecordRel` joins `lymdirs.DotLymDirName`, `lymdirs.ImplementerDirName` and `runplan.ConflictRecordName` with `path.Join` itself.
- **F4** (card 13): Edit group lists `plan:internal/weaverengine#Run`; an Intent bullet says `Run` calls `resumeStrands`, then `reenter` when `StopRecord` is set, after the run lock and the stale-conflict removal and before any batch.
- **F5** (cards 12, 13): `reenter` clears `StopRecord` and saves once the batch is recorded done or the verify re-run passes; each batch begin clears it under the lease.
  `TestRun_Reentry` gains the re-step-after-re-entry-then-plan-conflict row.
- **F6** (card 12): on a terminal `Await` result, `Run` sets `Record.Status` (and `DeadReason`), saves through `fixerStore.Save`, then sets `FixerState.Outcome`.
  `TestRun` gains the dead-first-fixer-then-second-fixer row.
- **F7** (cards 10, 12): card 10 creates `saveRunState`, which loads the file under the lease and merges each attempt's and fixer's `Delivered` before writing; `endAttempt` and every save of `Run`, `runBatch` and the resume go through it.
  `TestRun` gains the Delivered-survives-attempt-end row.
  This adds one helper so card 10 and card 12 share one merge, rather than restating it at each save site.
- **F8** (card 14): `Reset` writes `reset-pending` in `.lym/weaver/` before moving anything and removes it after the commit; step 2 commits only while the marker exists, otherwise refuses and commits nothing; the Intent states the bound.
  `TestReset` gains the no-marker and with-marker rows.
  The self-check widened this (see below).
- **F9** (card 21): `decisionNoticeInline` and `heldBackReason` re-signed with `weaverActive bool`; `addedNoticeLines` added under Edit; the Intent says `notifyLiveRuns` computes `weaverActive` once from `ActiveOrSetting`.
  The self-check widened this (see below).
- **F10** (cards 9, 10, 11, 12): dropped `TestClassifyAttempt`, `TestBatchGateSpec`, `TestEndAttempt`, `TestRunVerify` and `TestWriteSummary` with their Create bullets and paragraphs.
  `TestRun` gains every row the review listed, plus the rows those tests held that the review's list left out: a gate config lacking `card` or `batch` refuses; an unclean tree's paths recorded as `Uncommitted`; a report recorded twice appends one journal entry; a failed per-batch commit returns Stuck naming the re-step; a runner crash writes no verify-failure record; two verify failures write the record.
- **F11** (card 22): Edit gains `failuretrace#NewTracer`, `#Input`, `/doc.go`, and also `#Trace`, `#lastCardSHA`, `#TestTrace` and `#TestNewTracer`, since `Trace` reads `Input.State` and `lastCardSHA` takes the batch record (`websterengine.BatchState` is an alias of `runplan.BatchState`, `internal/websterengine/runplan.go:21`).
  `NewTracer` reads through `weaverengine.Active` with a nil `Setting`; `TestNewTracer` gains the planted-Weaver-state row.
- **F12** (card 1): Edit gains the motif fixture, `runplan/conflict.go` and `websterengine#ScratchDir`, and also `websterengine#Geometry`, whose `ScratchDir` field comment (`internal/websterengine/geometry.go:44`) lists the same two records.
  The Intent covers the fixture text, the conflict.go header with `ConflictRecordName`'s comment, and both ScratchDir comments.
- **F13** (cards 9, 11) — **fixed differently**: the fix put every doc and tripwire change in card 9.
  Card 9 takes the `card` and `batch` gate entries, and `planVerifySites` gains `batchGateSpec`'s file.
  Card 11 takes the plan-level verify's site, the `webster` key's Weaver row and `runVerify`'s file in `planVerifySites`.
  Reason: `cmd/lym/verifiedtree_test.go:81-84` fails on a listed site it does not find, and `runVerify` does not exist until card 11, so listing it in card 9 fails card 9's gate.
  Card 9's `batchGateSpec` now names the pause hold, which the tripwire's `Hold` check requires (`verifiedtree_test.go:75`).
- **F14** (card 23): the corrupt-Webster step sets `tapestry.implementer` to `weaver`.
- **F15** (card 12): `TestRun` gains the failed-final-commit row returning Stuck naming the re-step.
- **F16** (cards 1, 3): `lymdirs/doc.go` moved from Uses to Edit in card 3, with an Intent line; dropped from card 1's Uses.
- **F17** (cards 2, 20): `PATTERN-config-layers.md` moved from Extends to Edit in card 2; `loomcli/doc.go` moved to Edit in card 20, and the empty Extends group was dropped.
- **F18** (card 22): created `plan:internal/loomshed#ReworkPriorWeaver`; `archiveRunRecords` joins it onto `dest`'s parent, and `ArchivedRunRecordDirs` reads it.

## Self-check

- fix-claims, MEDIUM, card 21: `notifyLiveRuns` does not reach `addedNoticeLines`.
  Fixed with own change: `notifyLiveRuns` re-signed so `linesFor` takes `weaverActive`, `addedNoticeLines` re-signed with it, and `decisionVerb` and `decisionReplaceVerb` added under Edit.
  The lens offered a second option, having the verb compute `weaverActive` itself; that was not taken because it would compute the seam once per verb rather than once in the shared loop.
- fix-consistency, MEDIUM, card 11 (`planVerifySites` file unnamed); fix-proposal's out-of-lens note is the same finding, merged: fixed as proposed.
  Card 11 now says `runVerify`'s file declares `Verify`'s production value with `Hold:`, and that file is the listed site.
- fix-consistency, LOW, card 5's `StopRecord` description: fixed as proposed; it now says a batch begin and a successful re-entry clear it.
- fix-consistency, LOW, overview's card 22 index line: fixed as proposed; it names the failure tracer.
- fix-proposal, MEDIUM, card 14's stale marker: fixed as proposed.
  Step 2 removes a marker when nothing is pending; `Run`'s first batch begin removes any marker (card 14 adds `Run` and `TestRun` under Edit, with a `TestRun` row); `TestReset` gains the marker-with-nothing-pending row.
  The Intent names the one remaining window.
- fix-proposal, LOW, the reed wait-error `lost` verdict dropped from the tests: fixed as proposed in `TestRun`'s loss row.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
