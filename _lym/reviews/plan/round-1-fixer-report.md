---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Fixed as proposed for shed and batten. For fabric, the walk call goes into the merge-in verb's RunE in addMergeVerbs before MergeIn, not into websterInFlightWarnings: that function runs after MergeIn (internal/fabriccli/merge_verbs.go:146-149), so a conflicted merge would leave the worktree mid-merge and the walk would skip it. The shed row is a new tier1 test in internal/shedcli, since internal/loomcli cannot import shedcli."
  - id: F2
    disposition: fixed-differently
    reason: "The proposed snapshot before boardCriticalSection and restore after it run outside the board write lock, which boardCriticalSection takes and releases on return (internal/boardengine/board.go:103-107), so a concurrent board write could be overwritten. The snapshot is taken inside the mutation and the restore inside the afterSave wrapper, both under the lock. The proposed 'retried by the board's own sync' is not stated, since internal/boardengine/spawn.go shows no retry."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "The fake cannot return configreg.PrepareRunSettings' error: TestSeedChild_RunSettings is in package battenshed, and internal/configreg/configreg.go:16 imports internal/battenshed, so the call is an import cycle. A row of battencli's TestPrepareRunSettings pins the real moved-key text through Seed-Child's production adapter, and the battenshed row's fake carries that same text."
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

# Round 1 fixer-report — plan

## Changes

- **F1** (fixed differently), card 9.
  - **Edit:** gains `internal/shedcli#shedCLI.resolvePersistentPreRun`, `internal/battencli#battenCLI.arm` and `internal/fabriccli#addMergeVerbs`.
  - **Uses:** gains `internal/shedcli/doc.go` and `internal/fabriccli/doc.go`. `internal/battencli/doc.go` does not exist, so it is not listed.
  - **Create:** gains `TestShedVerbs_StaleStampReconcilesBeforeALoomRecipeArms`, a tier1 test in internal/shedcli. In it, `lym shed step` on a loom recipe over a stale stamp resolves `loom.code.max_bounces` to 2.
  - The Intent restates the method: the sites are every route to a strict loom or tapestry load, namely `LoadConfig`, `LoadTapestryConfig`, `loomcli.ArmAt`, `SettingLookup` and `configengine.Load` on either spec.
  - The Intent adds the shed pre-run, before `armFromSeed` for every verb, and `battenCLI.arm`, for every verb in place of `verb == "run"`.
  - It also adds merge-in's RunE, which calls the walk before `MergeIn` instead of inside `websterInFlightWarnings`.
- **F2** (fixed differently), card 8.
  - The `RewriteRunSettings` paragraph is rewritten from the code. The write saves and renders, and the detached sync commits and pushes; a board with no old key returns `noWrite`.
  - The restore happens under the write lock. The mutation reads `board.json`'s bytes, and the `afterSave` wrapper writes them back when the caller's hook fails.
  - A render failure leaves the moved `board.json` saved, and the retry then writes nothing.
  - A sync commit or push failure never reaches the walk.
  - The `EnsureFailingRunSettingsCarry` row asserts the entry is unchanged through that restore.
- **F3**, card 9: the walk sits after the skip of `loom`, `checklist` and `pusher` and before the `inReviewGroup` skip. The Bound states that the pusher, the checklist and the bare group run no walk.
- **F4**, card 9: `internal/hubreconcile/doc.go` moves from **Uses:** to **Edit:**. The Intent rewrites the opening line and the "Skips" retry line, and names the run_settings carry among the skip causes.
- **F5**, card 2: the PATTERN-config-layers paragraph adds the move-table exception to the line 7 rule and to the sentence at lines 77-78.
- **F6**, card 4: line 121 now says the `wantLocalDenied` loom and tapestry rows move to the nested paths.
- **F7**, card 6: `TestReconcile_FailureNamesItsFile` gains an unparseable `loom.yaml` row that fails in `moveOldKeys`, and its existing rows are unchanged.
- **F8** (fixed differently), card 5: a new row of `internal/battencli#TestPrepareRunSettings` is added under **Edit:**.
  - It pins the moved-key error from the real adapter: both full names and no `way forward:`.
  - The `TestSeedChild_RunSettings` row's fake carries that same text and asserts both names and exactly one `way forward:`.
- **F9**, card 8: the unreadable-board row writes a valid board config beside the unparseable `board.json`.
- **F10**, card 7: the write rule is now a byte difference after the move and the follow.

`lym loom validate-plan` reports ok, with only the existing informational `create-new-unit` finding on card 7.

## Self-check

Self-check changes made:

- Card 9's doc.go paragraph also rewrites "no start verb waits on a push". It also adds card 6's exception to the sentence that the walk removes no key the template lacks.
- Card 2 also names line 73 of PATTERN-config-layers.
- Card 8 renames `beforeCommit` to `afterSave` and the hook to `afterRunSettingsCarrySave`.

Fork findings:

- fix-consistency, LOW: doc.go:21-22 is not amended for the walk's moves. Fixed.
- fix-consistency, LOW: `beforeCommit` and `beforeRunSettingsCarryCommit` no longer match "runs as afterSave". Fixed by renaming to `afterSave` and `afterRunSettingsCarrySave`.
- fix-affected-set, MEDIUM: PATTERN-config-layers line 73 is a third stale member. Fixed in card 2.
- fix-affected-set, LOW: doc.go:21-22 is stale. This duplicates the fix-consistency LOW, and is fixed.
- fix-affected-set, LOW: doc.go:18 says "no start verb waits on a push". Fixed in card 9.
- fix-proposal, BLOCKING: F8's fake calling `configreg.PrepareRunSettings` is an import cycle. Fixed differently: the real text is pinned in battencli's `TestPrepareRunSettings`, and the battenshed fake carries it.
- fix-proposal, MEDIUM: the snapshot and restore in card 8 run outside the write lock. Fixed: both now run under the lock, inside the mutation and the `afterSave` wrapper.
- fix-proposal, MEDIUM: `websterInFlightWarnings` runs after `MergeIn`. Fixed: the walk is called in merge-in's RunE before `MergeIn`, and the Edit target is now `addMergeVerbs`.
- fix-claims, MEDIUM: the restore outside the lock. This duplicates the fix-proposal MEDIUM, and is fixed. The render failure after the hook is now stated.
- fix-claims, LOW: "the board's own sync retries it" is unsupported. Fixed: the card now says the carry neither sees nor retries a sync failure, and the moved `board.json` stays saved.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
