# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged onto one line, naming each lens and finding number.

## way-forward

- WF1 (BLOCKING), card 7, verdicts rebuilt from `Outcome` lose typed errors: fixed as proposed — card 7 keeps each action's error beside its outcome and builds the record and returned error from it (anchor way forward, conflict paths, takeover object returned as is).
- WF2 (MEDIUM), card 7, failed tracking-ref delete has no verdict: fixed as proposed — the delete runs as today outside `destroy.go`, a failure returns today's error with no record, and `TestPushSide` gains the row.
- WF3 (MEDIUM), card 1, unset-seam way forward names a Go call: fixed as proposed — the way forward names the test binary's steps, a hubforge hub or `hubforge.InstallRequiredSeams` (card 3 adds that export).
- WF4 (LOW), card 7, "returned before any lock" is false: fixed as proposed — the claim now says the refusal is returned under the push lock its caller holds, as today.
- WF5 (LOW) + RV6 (MEDIUM) + CV3 (MEDIUM) + GR2 (BLOCKING), card 7, the lease-lost row cannot reach the lease branch and is no `destroy.go` dirtiness refusal: fixed differently — the proposed fix, a reaching lease row, has no trigger point (`resetUnnamedTip` runs no push between its read and its reset, and a hook on the leased push never maps to `ErrLeaseRejected`, `internal/gitrepo/push.go:396-421`); the row is dropped, and card 7 records that this concern's only destructive action passes `dirtinessNA` (`pushclassified.go:457`), so D4's dirty-after-snapshot row lands with the teardown part, whose concern holds the first dirtiness-checked action (D9); card 11's live-pair row holds the criterion's other half.
- WF6 (LOW), card 9, the run-scope "no pair worktree" refusal names no way forward: disputed, every fix worse — adding a way forward changes the refusal's printed text, and D5 binds the move to change no message ("Every verb refuses, prints, writes and pushes the same as before its concern moves").

## row-validity

- RV1 (BLOCKING) + GR1 (BLOCKING), card 12 vs cards 9 and 11, `movesPair` without a run record breaks kept entry assertions and `TestBoltReplayers`' `ErrReplayerMissing` row: fixed differently — keeping the run record (one proposed option) needs the ledger and so the board replay D12 bars in fabric tests; instead the kept rows assert their branch outcome, `TestBoltReplayers` joins card 12 and writes its own run record through `writeStatus`, which takes its path from the installed seam via `RunStatusFileForTest`.
- RV2 (MEDIUM), cards 9 and 12, judge-only subtest both kept and moved: fixed as proposed — card 9 keeps the ref-unchanged half with Bolt left level; the diverged-Bolt half moves in card 12.
- RV3 (MEDIUM) + FE2 (MEDIUM), card 12, unlisted helpers for moved tests and dead fabric helpers: fixed as proposed — card 12 creates `reconcileHub`, `newReconcileHub`, `writeRunStatus`, `teardownRecorder` and `requireEntry` in `boardengine_test`, edits `writeStatus` and `newFailureFixture`, and deletes `endedTemplate`, `arrangeEnded`, `endedHub`, `writePairStatus`, `publishBoard` and `originBoard`.
- RV4 (MEDIUM) + CV10 (MEDIUM), card 5, `NoticeTags_*` rows deleted with no successor: fixed as proposed — `TestMergeFailedPush` gains the `FailedPushNotice` rows taken from them.
- RV5 (MEDIUM) + CV6 (MEDIUM), cards 3, 4, 11, the run-lock and codec adapters are untested in the supplying module, and `TestOriginCodeTip` asserts the codec's verdict: fixed as proposed — `TestInstallRequiredSeams` checks all three adapters against shed (status path, codec machine, `shedrun.PrimeRunLock` held), and `TestOriginCodeTip` drops its `Machine` assertions.
- RV7 (LOW), card 11, the dirty-in-verdict-hook row repeats the kept dirty row: fixed as proposed — the row is dropped.
- RV8 (LOW) + CV11 (MEDIUM), card 6, deleting the engine's `TestDescribePushLockHolder` leaves the file mapping untested: fixed as proposed — it keeps its `NoRecord` and `Corrupt` rows.
- RV9 (LOW), card 7, the deferral sentinel is undeclared: fixed as proposed — `errRetryDeferred` is a `Create` handle and `TestOutcomeClassOf` gains its row.
- RV10 (LOW), cards 2 and 4, "shed's own writer" names no writer: fixed as proposed — the rows write through `state.UpdateJSON` over shed's `Status`, the write every shed status writer makes.

## coverage

- CV1 (MEDIUM), no card carries the `origin`, `remoteonly` and `pendingflags` rules: fixed as proposed — card 4 adds `MarkBeamVerdict`, `ReadOriginForVerdict`, `RemoteOnly` and `TestOriginRecordRules`; card 6 adds `FlaggedLoopNext` and `TestFlaggedLoopNext`.
- CV2 (MEDIUM), no tier1 seam test asserts a flow's snapshot: fixed as proposed — card 7 adds `TestPushClassified_Snapshot` (push flow, shared by card 8's loop) and card 9 adds `TestReconcileOrigin_Snapshot` (reconcile flow).
- CV3: merged with WF5 above.
- CV4 (MEDIUM), mutation-array rows missing for the coalescing push and the teardown's entries: fixed as proposed — card 7 widens `TestCoalescePushBothAt_AdvancesBothSidesAndLeavesNoWarpRootLock` to the exact array, which `CoalescePushRequest` shares; card 12's `TestReconcileOrigin_EndedRuns` asserts the exact array with the teardown's entries.
- CV5 (MEDIUM), `OriginCodeTip`'s refusal reached by no row: fixed as proposed — `TestInstallRequiredSeams` asserts both seams' refusals of `OriginCodeTip`.
- CV6: merged with RV5 above.
- CV7 (MEDIUM), `PushSide` branches with no row: fixed as proposed — `TestPushSide` gains a row per listed branch.
- CV8 (MEDIUM), deleted push rows pin engine reads: fixed as proposed — card 7 keeps `ARejectionOfABranchWithNoUpstreamWhoseNameOriginAlreadyHoldsIsADivergence`, the `Code` and `RecordsWithNoUpstream` rows, and `ThePrimesCodeBranchPushedAfterAnOriginRewindIsPushedAndRecordsNothing`.
- CV9 (MEDIUM), card 8 drops the unclassified-lookup branch: fixed as proposed — the verdict returns it unclassified and `TestRecordsSideLanding` gains the row.
- CV10: merged with RV4 above.
- CV11: merged with RV8 above.
- CV12 (MEDIUM) + GR3 (MEDIUM), card 12 deletes the snapshot-tag and `OwnerOf` rows whose claimed holders cannot reach the engine reads: fixed as proposed — the moved tests keep the earlier-snapshot-tag row and the nil-`OwnerOf` and `OwnerOf`-error rows.
- CV13 (MEDIUM), D2's "no more git processes" for the push side argued without a test: fixed differently — the proposed git-call count has no seam to count through (`internal/gitexec` exposes no call hook); `TestPushClassified_Snapshot`'s second row asserts that a branch with no upstream reads no origin phase and no judgment, the reads that would add processes.
- CV14 (LOW), the new readers' ref-storage fallback is unstated: fixed as proposed — card 7 states the per-verb readers make today's calls unchanged, so their ref-storage handling is today's.
- CV15 (LOW) + GR5 (LOW), card 9 refuses a run scope for an unset run-locks seam it never uses: fixed as proposed — only a moving hub scope needs `SeamRunLocks`, and `TestOriginJudge`'s row covers a run scope.
- CV16 (LOW), a fabric test building its hub without hubforge would meet the refusal: fixed as proposed — card 3 adds `hubforge.InstallRequiredSeams` and names the search for the test files that must call it.

## forced-edits

- FE1 (BLOCKING), card 4, re-signing `originStatusAtTip` breaks its part-4 callers: fixed as proposed (second option) — `originStatusAtTip` keeps its signature and moves to `handover.go` as a decode over the new `originStatusBytesAtTip`, which refuses an unset seam.
- FE2: merged with RV3 above.
- FE3 (LOW), the `fetchMovedHeads` allowlist reason goes stale: fixed as proposed — card 9 edits `cmd/lym#remoteRefsLockFetchAllowlist` with the new reason, and card 11 drops its claim and `Uses` entry.
- FE4 (LOW), card 7 leaves `goneEntryEnded`'s file unstated: fixed as proposed — it stays in `pushclassified.go`, and `TestBoltReplay_FabricNamesNoBoardFile` is under `Uses`.
- FE5 (LOW), card 10 `Uses` a test card 12 deletes: fixed as proposed — the entry is dropped.
- FE6 (LOW), `ErrPushRetriesExhausted` changes class in `outcomeClassOf`: fixed differently — the proposed re-check of `TestRunRule`'s rows needs no change, since the error is a push error no snapshot read returns; card 7 states that and lists `TestRunRule` under `Uses`.

## guard-reach

- GR1: merged with RV1 above.
- GR2: merged with WF5 above.
- GR3: merged with CV12 above.
- GR4 (MEDIUM), card 11's dirty row reaches `ffWorktree`, not `destroy.go`: fixed as proposed — the row is dropped (with RV7), and D4's dirty row is placed as WF5's line records.
- GR5: merged with CV15 above.
