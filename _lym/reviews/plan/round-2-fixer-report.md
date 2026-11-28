---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: ruling
    reason: "Parent decision 'A protected refusal of the leased reset keeps today's other record' (option a) carried out in card 7."
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed-differently
    reason: "The holder of the records write lock is not added to the kit, because no moved row takes that lock: the rows that call holdWeftWriteLock stay in fabric. Fabric's requireEntry also joins pushkit, because otherwise it would be a second copy in boardengine_test. tagsOf, landOn and originHasBranch become gitkit calls inline, since gitkit already exports what they need. For boardFileOf, boardkit gains BoardFile, the encoder WriteBoard already uses, and the entry builders are dropped for the entry JSON strings WriteBoard takes."
---

# Plan fixer report, round 2

## Changes

- **F15** (`_lym/plan/07-push-side-core.md`, `_lym/plan/03-required-seam-installer.md`, `_lym/plan/00-overview.md`)
  - Card 7's Edit list gains `TestCoalescePushRequest_CoveredCallDefers` and `TestCoalescePushBothAt_HeldByAPauseWithoutThePushLock`, and its Uses list gains `SetRunRecordPaths`.
  - A new Tests bullet says each of the two tests sets the run-records paths seam through `SetRunRecordPaths`, with paths under its own temp repos. Each restores the seam and does not call `t.Parallel`. The bullet also explains why package `fabricengine` cannot use hubforge's installer.
  - Card 3's search paragraph now names `pushdeferral_integration_test.go`, which card 7 handles, and `commit_partial_integration_test.go`, which only stubs the spawn. The claim that the search finds no such file is gone.
  - The overview's sequential-seam decision said every seam-setting test builds a hubforge hub first. It now follows D13's wording, "one that also builds a hub … builds it first", so the two hubless tests agree with it.
- **F16**: covered under Rulings.
- **F17** (`_lym/plan/12-board-test-moves.md`)
  - The `statusJSON` Create line and its fixtures mention are removed.
  - `writeRunStatus` now takes `history ...string` and writes shed's `Status` and `HistoryEntry` types through `state.UpdateJSON` at `shedrun.StatusFile`, under `shedrun.StatusLock`.
  - The card says the `APrimeWeft…` row writes its pair record and its prime record that way and commits the prime record.
- **F18** (`_lym/plan/12-board-test-moves.md`, fixed differently; the frontmatter reason gives the departures)
  - Removed these Create lines: `reconcileHub`, `newReconcileHub`, `failureFixture`, `newFailureFixture`, `pairFixture`, `newPairFixture`, `pushPair`, `newPushPair`, `fakePushNotifier`, `bareBranchTip`, `originHasBranch`, `requireOneFailedPush`, `requireNoFailedPush`, `failedPushesOf`, `countingBackoff`, `holdWeftWriteLock`, `landOn`, `tagsOf`, `boardEntry`, `withBoardTitle`, `withBoardStatus`, `boardFileOf`, `boardTitles` and `boardengine_test#requireEntry`.
  - Added the `pushkit` Create lines `RequireOneFailedPush`, `RequireNoFailedPush`, `CountingBackoff` and `RequireEntry`, plus the `boardkit#BoardFile` Create line and a `boardkit#WriteBoard` Edit line.
  - Added plain functions over the hub: `newReplayHub`, `pushPrimeRecords`, `commitOnOrigin`, `endEntryOnOrigin` and `addPushedPair`.
  - Added Delete lines for fabric's `requireOneFailedPush`, `requireNoFailedPush`, `failedPushesOf`, `countingBackoff`, `requireEntry` and `requireTrackingRefPrunedWithUpstreamSet`. The last moves because its only callers are the moved rows.
  - Added Edit lines for every fabric test and helper whose calls retarget, named by member, since the validator refuses a `_test.go` file under Edit.
  - Moved rows pass no `Notifier` and drop their mail assertions.
- Facts the review left unverified, now read:
  - In a binary that installs no notifier, `installedNotifier` returns nil and `recordPushFailure` writes the record and mails nothing (`internal/fabricengine/failedpush.go:38`, `:99`). Nothing in `internal/boardengine` installs a notifier.
  - `gitkit` exports `RevParse`, `BranchExists`, `CommitFile` and `Git` (`internal/gitkit/query.go:75`).
  - `boardkit` had no encoder that returns content, which is why `BoardFile` was added.
  - Kits may import `internal/fabricengine` but not `gitkit`, `gitexec` or `hubforge` (`internal/testkit/enforcement_test.go`'s `kitDeniedImports`).
  - No validate-plan `--overlay` flag exists: `lym webster validate` refused that flag.
- Plan validator, `lym webster validate`:
  - Its first runs refused the `_test.go` file lines under card 12's Edit and a method glyph written without its receiver. Both are fixed.
  - It now reports one blocking finding, `plan-unapproved`, the frontmatter's `approved: false`, which approval sets.
  - It also reports four informational `create-new-unit` findings for the new `pushkit` package.

## Self-check

- fix-consistency, LOW, the reconcile rows had no named hub builder: fixed. Card 12 now says those rows build on `newReplayHub`, `hubforge.AddPair` and `pushPrimeRecords`.
- fix-consistency, LOW, the ImpactSummary said "no production code changes" next to a kit edit: fixed. It now reads "no package outside the test kits and test files changes".
- fix-affected-set, LOW, `BareBranchSHAForTest` in the moved `ALandingCommit…` row was not named: fixed. Those reads become `gitkit.RevParse`.
- fix-affected-set, LOW, branch names were missing once `addPushedPair` returns only a location: fixed. Rows read each branch through `gitkit.CurrentBranch` of the hub accessors.
- fix-claims, MEDIUM, the "no second copy of a helper fabric keeps" claim was false: fixed. The sentence now says only that a helper using neither git nor hubforge lives in one kit, and that git-using helpers fabric keeps are restated as plain functions, since kits may not import gitkit. `requireEntry` moves to `pushkit`, and fabric's copy is deleted.
- fix-proposal, MEDIUM, the retarget in three other concerns' files departs from D9's own-files criterion: fixed in the card by stating the departure and its cause. Whether to accept it is deferred to the parent, below.

## Disputed

None.

## Deferred

- **The D9 file criterion against the F18 retarget.** Card 12 now renames call sites in `landing_integration_test.go`, `boltpullfirst_integration_test.go` and `rejectedtakeover_integration_test.go`. These files belong to parts 4 and 5.
  - Acceptance criterion: "each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types (D9)".
  - `PATTERN-test-isolation` requires shared support to live in one kit, which forces the rename once the helpers' definitions move to `pushkit`.
  - Choosing between those two rules is the parent's call: accept the rename as an exception, or keep local copies in those three files until their parts.

## Rulings

- **F16**: the parent decision "A protected refusal of the leased reset keeps today's other record" (option a) is carried out in card 7:
  - Create line `plan:internal/fabriccore#OutcomeProtectedUnrejected` -> `const OutcomeProtectedUnrejected OutcomeClass = "protected_unrejected"`.
  - `outcomeClassOf` maps a protected refusal that does not wrap `gitrepo.ErrPushRejected` to `OutcomeProtectedUnrejected`, such as a hook's refusal of `resetUnnamedTip`'s leased reset. A protected refusal that wraps it stays `OutcomeProtected`.
  - Either protected class stops the loop. The unrejected outcome's error is not joined to the rejection, and `JudgePush` records `other` with that error and asks no `FieldPushJudgment`. The judgment read now applies only to an `OutcomeProtected` refusal of the push itself.
  - `protectedParentWayForward` keeps today's condition for either class, as `pushJob.record` sets it.
  - Added a `TestOutcomeClassOf` row and a `TestPushSide` row.
  - The ruling was neither mistaken nor redundant, and I did not message the parent about it.
