```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the 12-card plan (whether each planned test row can do what its card says, and whether it is needed):

1. **BLOCKING — Card 12 against cards 9 and 11 (kept `TestReconcileOrigin_Moves` rows).** Card 12 makes `movesPair`'s template write no run status, but the rows cards 9 and 11 keep still assert entry states.
   - The template writes the record at `internal/fabricengine/originreconcilemoves_integration_test.go:134` (`writeStatus(tb, originHubOf(tb, h), "task", "Land")`).
   - Kept rows assert an entry through `requireEntry`: the clean/dirty pair-weft row (`:227`, `:243`), the live-pair row (`:348`, `:359`, `:369`), the held weft lock row (`:530`) and the held Bolt push lock row (`:647`).
   - `classifyEntries` reports entries only for recorded pairs (`recordedPairs`). With no run status those assertions fail. They include the live row's `EntryStopped` with `liveWayForward`, which D4's live-pair criterion rests on.
   - Keeping the record makes `recordedRuns` non-empty, so `readOriginEnded` calls `addLocalCreatedStamps`, then `runLedger`, then `ErrReplayerMissing` once `boardkit.InstallReplay` is dropped.
   - No card says which of the two the kept rows take. The split leaves a fact for the implementer to find out at run time; card 12 says so itself ("a kept row that still reads the ledger is a defect of this card's split").

2. **MEDIUM — Cards 9 and 12 (`TestReconcileOrigin_Judge` judge-only row).** One subtest is both kept and moved.
   - The subtest is `internal/fabricengine/originreconcile_integration_test.go:523`, "A judge-only reconcile fetches a moved branch's objects and leaves every ref unchanged, and reads a diverged Bolt as needs_replay". The needs_replay assertion is at `:537`.
   - Card 9 keeps it as "the judge-only fetch leaving every ref unchanged". Card 12 moves "needs_replay of a diverged Bolt" to `TestReconcileOrigin_OwnedElsewhere`.
   - A diverged Bolt makes `readOriginEnded` call `localLedgerRuns`, which needs the ledger. With `InstallReplay` dropped at `:220`, the kept half fails unless Bolt stops diverging. Neither card says it does.

3. **MEDIUM — Card 12 (helpers the moved tests need).** The three new `boardengine_test` tests need fixtures that live only in package `fabricengine_test`, and no card lists them.
   - Fixtures from the origin test files: `originHub`, `newOriginHub`, `originPair`, `otherMachine`, `originCommitFile`, `writeStatus`, `requireEntry`, `entryOf`, `moveReconcile`/`moveReconcileWith`, `kindCount`, `endedTemplate`/`arrangeEnded`/`endedHub`, `publishBoard`/`originBoard`, `writePairStatus`.
   - Fixtures for the push rows: `boardEntry`/`withBoardStatus` (`replay_integration_test.go`), `recordsFixture`, `newFailureFixture`, `pairFixture`, `fakePushNotifier`, `countingBackoff`, `holdWeftWriteLock`.
   - None is declared as a `Create` handle on card 12, nor is "their own recorder" that replaces `teardownRecorder`.
   - Helpers whose last fabric users move or are deleted (for example `endedTemplate`, `arrangeEnded`, `writePairStatus`) are left dead in fabricengine with no `Delete`.

4. **MEDIUM — Card 5 (`NoticeTags_*` deletion).** The deleted rows have no successor.
   - The five `TestFailedPushRecord` `NoticeTags_*` rows are deleted as "held by the core rows above".
   - None of `TestFailedPushWayForward`, `TestMergeFailedPush` or `TestFailedPushFilters` has a row for `fabriccore.FailedPushNotice` (line, topic, run from `PairSlug`). The notice's run and topic go untested, against D5's rule that a test goes only with its successor.

5. **MEDIUM — Cards 3, 4 and 11 (adapter tests).** Two of the installer's three adapters have no test in the module that supplies them.
   - `TestInstallRequiredSeams` (card 4) asserts only the run-records path adapter, through `OriginRunStatus`.
   - The run-lock adapter (`shedrun.PrimeRunLock`) is held by no test: card 11 drops the prime-lock row's use of `shedrun.PrimeRunLock`, and `TestReconcileOrigin_PrimeLockHeld` sets its own lock path.
   - The codec adapter's wiring (`LastMachine` to `shedengine.StatusLastMachine`) is reached only indirectly in fabricengine. `internal/fabricengine/codetip_integration_test.go:60` asserts `Machine: "oslo"` over the hubforge-installed seams. That is a fabric test asserting the codec's verdict, which D13 bars ("asserts no verdict of it").
   - D13's acceptance criterion asks for each adapter to be tested in the supplying module. Rows for the lock path and the machine belong in `TestInstallRequiredSeams`.

6. **MEDIUM — Card 7 (new `AnOriginTipMovedAfterTheReadLosesTheLease` row).** The row may not reach the branch it claims.
   - The leased reset reads origin's tip from the fetched upstream (`resetUnnamedTip`: `j.repo.UpstreamSHA()` after `Fetch`). A pre-receive hook on origin fires only on a push.
   - So "another clone's push landing between the read and the leased reset" has no trigger point. The only push in that gap is the leased one itself, whose own hook-side ref move gives an error the card does not show maps to `gitrepo.ErrLeaseRejected`.
   - The refusal is git's lease, not one of `destroy.go`'s containment, ownership or dirtiness checks (`internal/fabricengine/destroy.go:3`). The row does not hold D4's "target turned dirty, refused by destroy.go's checks" criterion as the card claims.

7. **LOW — Card 11 (new dirty-in-verdict-hook row).** The row repeats an existing one.
   - The judging snapshot never reads dirtiness: neither `MoveFacts` nor `BranchFacts` carries it, and `ffWorktree` reads it live at move time.
   - Dirtying the checkout inside the verdict hook therefore takes the same path as the kept row "stopped as dirty otherwise" (`originreconcilemoves_integration_test.go:215`). It is not the destroy.go after-snapshot refusal D4 names.

8. **LOW — Card 6 (engine `TestDescribePushLockHolder` deletion).** One branch loses its test.
   - The engine now maps an absent file (`os.ErrNotExist`) to `found=false` and a decode failure to `readErr` before calling the core.
   - The kept `TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld` covers only a present, live record. The engine's no-record and corrupt-file mapping goes unreached.

9. **LOW — Card 7 (deferral sentinel).** "A retry that reported a deferral returns an error the runner wraps so it reads `OutcomeDeferred`" needs a new sentinel error or type that `outcomeClassOf` matches. It is not declared as a `Create` handle, and `TestOutcomeClassOf`'s planned rows do not name it.

10. **LOW — Card 4 (`TestInstallRequiredSeams` writer).** The test writes "with shed's own writer", but shed has no named status writer, only `state.UpdateJSON` call sites inside shedengine. The test's direct `state.UpdateJSON` is not shed's writer, so the adapter is not checked against data shed itself wrote.

Tool calls made:
- `lym quarry resolve` of `copyTemplate`, `hubRunReader`, `remoteRefsLockFetchAllowlist`, the test shims, `movesPair` and `TestOriginCodeTip`: to confirm the named glyphs exist, which my context had not checked.
- grep of `shedengine/status.go` History and Machine, the codetip test's Machine assertions, and `destroy.go`'s lease and dirtiness lines: to check the codec row and the lease row, which my context had not read.
- grep of the Moves subtests' `requireEntry` and `writeStatus` lines and the Judge judge-only Bolt row: to check whether the kept rows still reach their entries without a run record or replayer, which my context had not read.
