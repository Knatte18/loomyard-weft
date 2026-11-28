```yaml
lens: affected-set
outcome: reported
```

**Lens: affected-set.** For each change the cards make, I listed what reads or asserts it and checked which cards cover it.

**Per change: the set found and the cards covering it**
- `OriginRunStatus` re-signed to bytes. Set: `pairteardown/runread.go:129` and `originreads_integration_test.go:65-70`. Card 4 covers both.
- `originStatusAtTip` moved onto `originStatusBytesAtTip`, which now refuses an unset seam. Set: `handover.go:200,215` (`OriginParentRunStatus`, `OriginBattenStatus`); `takeover.go:95` (`ReadUpstreamTakeover`, reached from `retryWeftPush`'s `takeoverStop`); their tests `takeoverrun_integration_test.go`, `rejectedtakeover_integration_test.go`, `handover_integration_test.go`. Card 4 covers only the move. No card covers the new refusal's reach, and card 3's test search misses these callers (F1).
- `readCodeTipAt` reads the machine through the codec. Set: `TestOriginCodeTip`, which card 4 covers, and `plantRunTransition` in `codetip_integration_test.go`, which still imports `shedrun` and `shedengine`. No card covers `plantRunTransition` (F2, F6).
- `tryPrimeLock` takes its path from `RunLocks`. Set: the `originreconcilemoves_integration_test.go:379` row, which card 11 covers, and the `uncontainedWriteAllowlist` entry at `cmd/lym/uncontainedwrite_test.go:72-73`, whose reason card 11 wrongly calls still true (F4).
- `outcomeClassOf` gains five classes. Set: `TestOutcomeClassOf` (card 7); `runRule`'s transport-failure recording (card 7 argues it holds); the class list in `internal/fabricengine/doc.go:652`, which card 7 reaches only through Extends (F5).
- `fetchMovedHeads` moves into the moved-heads read. Set: the `remoteRefsLockFetchAllowlist` entry at `cmd/lym/remoterefslock_test.go:22`. Card 9 covers it.
- Engine functions deleted or moved: `entryFacts`, `classifyEntry`, `mergeEndedEntries`, `needsMove`, `unionRuns`, the retry loops and the `localBranch` methods. Set: `EntryFactsForTest` (`export_test.go:964`) and `ClassifyEntryForTest`, which card 9 covers. The way-forward shims in `export_test.go:934-947` keep compiling, because the engine functions stay as delegations. Nothing is missed.
- The board-file readers map in `TestBoltReplay_NamesNoBoardFile` (`boltreplay_test.go:176-181`) names `goneEntryEnded`, `addLocalCreatedStamps` and `readOriginOwners`. Cards 7, 9 and 11 keep each where the map expects it. Covered.
- The tests card 12 moves to `boardengine_test`. Set: the fixtures they call from `fabricengine_test`, which card 12 covers only in part (F3).
- The `TestPersistentPreRunE_InstallsSeams` names list. Card 3 covers it.

**Findings**

F1. MEDIUM, consistency, `_lym/plan/03-required-seam-installer.md` (Intent, the grep line).
Card 4 makes `originStatusAtTip` refuse an unset `SeamRunRecordPaths`, so the refusal also reaches `OriginParentRunStatus`, `OriginBattenStatus` and `ReadUpstreamTakeover`, and through `takeoverStop` the retry of every records push. Card 3's search for tests that would meet the refusal greps only `ReconcileOrigin|OriginRunStatus|OriginCodeTip`. It also says only cards 4, 9 and 10 read the seams.
- Missed members: `handover.go:200,215`, `takeover.go:95`, `pushclassified.go:988` (`takeoverStop` in `retryWeftPush`), and the tests that reach them: `takeoverrun_integration_test.go`, `rejectedtakeover_integration_test.go`, `handover_integration_test.go`, and the push-retry rows of `pushanchored_integration_test.go` and `pushclasses_integration_test.go`.
- Evidence: the card 3 grep line; `takeover.go:95`; `handover.go:200,215`.

F2. MEDIUM, consistency, `_lym/plan/12-board-test-moves.md` (Intent, first line).
Card 12 says this concern's fabric tests stop importing `internal/shedrun` and `internal/shedengine`. Yet `codetip_integration_test.go`, the test of `OriginCodeTip` that card 4 edits, keeps both imports through `plantRunTransition`. That helper calls `shedrun.TreeRunFileRels` and marshals a `shedengine.Status`, and the pair-push owner-guard rows share it. No card rewrites the helper or names the file.
- Evidence: `internal/fabricengine/codetip_integration_test.go:15-16,22-39`.

F3. MEDIUM, design, `_lym/plan/12-board-test-moves.md` (the fixtures `boardengine_test` gets).
The rows moved into `TestPush_BoardReplay` call `fabricengine_test` helpers that the new package cannot see. The card creates only `reconcileHub`, `newReconcileHub`, `writeRunStatus`, `teardownRecorder` and `requireEntry`.
- Missed members: `newFailureFixture` and `newFailureFixtureAt`; `recordsFixture`, with its `boardBranch` and `originCommit` methods; `fakePushNotifier` and its `options`; `requireOneFailedPush`; `requireNoFailedPush`; `bareBranchTip`; `boardEntry`; `boardFileOf`; `boardTitles`; `withBoardTitle`.
- Evidence: `pushclasses_integration_test.go:45-83`, and the calls inside the `ABoltPushRejectedOverAnUnrelatedOriginEntryIsReplayedAndLands` row.

F4. MEDIUM, consistency, `_lym/plan/11-origin-moves-core.md` (Uses `cmd/lym#uncontainedWriteAllowlist`; the Intent says the entries "stay true").
After card 11, `tryPrimeLock`'s `os.MkdirAll` path comes from the `RunLocks` seam, which a test sets to a path of its own (card 11's `TestReconcileOrigin_PrimeLockHeld`). The allowlist reason "the path is shedrun's, never caller-derived" is then false. D7 asks that an allowlist naming a moved symbol move with it in the same change, and card 11 lists the entry under Uses only.
- Evidence: `cmd/lym/uncontainedwrite_test.go:72-73`; `originreconcilemoves.go:44-47`.

F5. LOW, consistency, `_lym/plan/07-push-side-core.md` (Extends `internal/fabricengine/doc.go`).
Card 7 adds five outcome classes to `outcomeClassOf`. The engine doc enumerates the mapping as "done, anchor stop, auth refused, rejected, destructive refusal, transport failure or failed", so the card must rewrite that sentence, but it reaches `doc.go` only through Extends.
- Fix: list the doc under Edit.
- Evidence: `internal/fabricengine/doc.go:651-652`.

F6. MEDIUM, consistency, `_lym/plan/04-origin-reads-seams.md` (the `TestInstallRequiredSeams` codec check).
The card plants the transition "as `TestOriginCodeTip` plants one". The planting helper `plantRunTransition` and its `pairFixture` live in package `fabricengine_test`, which `internal/fabriccli_test` cannot import. The card names no planting helper of the test's own.
- Evidence: `codetip_integration_test.go:22-39`; card 04's Tests bullet.

Tool calls:
1. A repository grep for docs and code naming the moved or deleted symbols, to find readers outside the manifest.
2. A grep of `internal/fabricengine/doc.go` and `pattern/`, `docs/`, `contracts/` for the class list and the concern's files, to find doc readers.
3. The hub builders of every test file reaching a seam-reading verb, to size card 3's search set.
4. A grep of test files and `export_test.go` for the deleted symbols, plus the readers map in `boltreplay_test.go:174-195`, to find shims and scans naming them.
5. The helpers one moved `pushclasses` row calls, to check card 12's fixture list. That came from the parent's earlier call, and this fork used 4 calls.
