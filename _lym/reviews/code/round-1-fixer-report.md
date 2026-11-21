---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Deleted the Bolt weft:main marker fallback, the weft conclude arm and weft recorded-merge guard, the empty-source exemptions and drift's partial-scope translation with ScopeWasPartial, with their doc text and tests. Did not delete the stored ScopeRecord partial readers in internal/driftengine/verdict.go:102, check.go:279 and records.go:20: they read a task's recorded scope on the board, a different datum from the config key the review's translation text names, and removing them changes the check's skip semantics, which the review's fix does not ask for."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The review proposed a row in TestAnchorChange. BeginAnchorChange refuses a staged deletion as an uncommitted change (internal/fabricengine/anchorchange_integration_test.go run: 'holds uncommitted or untracked changes'), and the deletion is staged only by Add's moveForkedWeftToAnchor (addanchor.go:84). The test is TestAdd_AtAnotherAnchorCommitsTheDeletionOfTheOnlyTrackedRunRecords in add_runrecords_integration_test.go instead, asserting the fork-to-branch diff carries the deletions and nothing stays staged."
  - id: F6
    disposition: fixed-differently
    reason: "The review proposed rows in TestExcludeLinesToStrip, a no-git unit test in package fabricengine that cannot build hubs. The sibling walk is covered by TestUnwireJunctions_ExcludeLineFollowsTheSiblingWalk in the new unwire_exclude_integration_test.go: a same-anchor link keeps the line, another anchor and a plain directory let it go."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "Set the door to one-way. The Merge danger section is two strict lines the summary parser reads, so the explanation of the older-binary draft_status and retired-key load change is a bullet above it in _lym/landing/summary.md. That file is under _lym, so it is edited and not committed."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: deferred
    reason: "Done: configreg.go:174 (Lookup of every shuttleengines.ConfigModules name in TestModules_SeedOnlyAndHubWideFlags) and fabric.go:742 (warp_binding_detail present for the unbound hub in TestCommand_ReconcileLeavesAMissingWarpBindingUnrecorded). Not done this round: configsync.go:482 (needs a models.template-base seed field in the TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate table loop, 440 lines of which I did not read), recordsgit.go:92 (the exact exclude content depends on git init's template header, which I did not measure) and circling.go:74 (I did not find which readCirclingDecision result carries the automatic flag). Each needs a read of its test loop before an assertion that fails with the mutant can be written."
---

## Changes

Commits, in order, each green under `lym gate test` for the packages it touches (tier 0 across the changed packages, tier 1 for fabricengine, hubreconcile, fabriccli, configsync, batcher, boardcli, driftengine, loomengine, configreg, boardengine):

- F1: `//testtiming:keep` reasons above the seven contained tests, each naming the assertion the covering tests lack.
- F2: `loomengine.ConfigSpec` retires the thirteen plan keys as `guard` keys with a way forward naming the tapestry module, in the `shedOnlyKeys` style; the `a loom plan_max_bounces is refused` row asserts that way forward.
- F3, F4: `recordedAnchorOnWeft` reads no Bolt marker; `concludeMergeSides` and `recordedMergeGoneReason` act on the warp side alone; the empty-source exemptions, their doc.go and mergestate.go text, drift's `ScopeWasPartial` and partial-to-full translation are gone, and `scope: partial` is refused naming full.
  The stale `.fabric-anchor` refusal names the prime's weft worktree, the records branch and the retry; `TestCloneHub_StaleFabricAnchorHardErrors` applies the remedy and asserts the retried clone adopts `backend`.
- F5, F6, F7: the three new tests named above and the `TestEnsure_FailedWalkAfterAReinterpretationLeavesNoMarkerAndTheNextWalkCommitsItOnce` serial test.
- F8: sentences describing deleted migrations are removed from `docs/overview.md`; `migrateRunSettings` is `validateRunSettings`; the `fabric.go` comment and the discussion stencil aside are corrected.
- F9: door set to one-way in `_lym/landing/summary.md` (not committed, it is under `_lym`).
- F10: `retiredKeyError` offers the reconcile verbs only for a template-named profile; `TestActive` gained an operator-named profile row.

## Disputed

- Self-check fork finding on F1 (fix-proposal, LOW): premise false. The keep reasons name assertions in each test body I read: `TestDraftStatus`'s table rows in `internal/boardengine/draft_test.go:12` onward are the direct test of a new task reading draft, `TestPromotePreservesRecipeAndLabels` and `TestBoardRefusesUnconfiguredLabel` were read whole in `internal/boardengine/board_test.go`.
- Self-check fork finding on F3 (fix-proposal, LOW): premise false. Every production write of a merge record sets `WarpSource`: `internal/fabricengine/merge.go:265` and `internal/fabricengine/merge.go:479`, so an empty source reaches the guard only through a hand-written record.

## Deferred

- F11, the three sub-items named in its disposition above.
- Self-check fork finding on F2 (fix-proposal, LOW): this hub's own `_bolt/_lym/config/profiles.yaml` still holds `loom.plan_*` keys. That file is outside this worktree, so the fixer does not edit it; with F2 fixed, a tuned value there is refused with a pointer to tapestry.

## Gate repair

The comment-style gate refused the round for fixed-column wraps and over-long lines in eight comments.
Each is now split at sentence boundaries, with no change to code or tests:
`batcher/doc.go`, `add_runrecords_integration_test.go`, `clone.go`, `clone_adopt_test.go`,
`mergein_recovery_integration_test.go`, `mergelifecycle.go` and `unwire_exclude_integration_test.go`.

## Reach notes

The gate listed the existing tests that reach each changed function and named no function without a reaching test.
Every changed function it lists, such as `retiredKeyError`, `templateNamesProfile` and `validateRunSettings`, has reaching tests, and the round's added tests are among them.
No test was added and no code changed in answer to it.

## Rulings

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: `recordedMergeGoneReason` still evaluated the weft side. Fixed, with its doc.
- fix-consistency, LOW: `doc.go` text on the deleted weft arm. Fixed.
- fix-consistency, LOW: `mergestate.go` comment on empty sources. Fixed.
- fix-proposal, MEDIUM: the F4 refusal named the Bolt worktree for a commit on the records branch. Fixed by naming the prime's weft worktree.
- fix-proposal, LOW: F3 empty source raises a raw git error. Disputed, premise false.
- fix-proposal, LOW: F8 left sentences at `docs/overview.md` 1526-1528. Fixed differently: they describe the generic prune `lym config reconcile --apply` still performs (`pattern/PATTERN-config-layers.md:73`), so they stay, and the first now names reconcile as its subject.
- fix-proposal, LOW: F1 keep reasons unchecked. Disputed, premise false.
- fix-proposal, LOW: F2 hub data. Deferred, outside the worktree.
- fix-affected-set, LOW: `mergestate.go:35` comment. Fixed.
- fix-affected-set, LOW: `verdict.go` partial case. Recorded under F3's `fixed-differently` reason.
