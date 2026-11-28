```yaml
lens: coverage
outcome: reported
```

Coverage lens on the part-2 fabric plan: I checked every decision, every acceptance criterion and every branch the cards introduce, and looked for the row that reaches each one.

**Findings**

1. **MEDIUM** (Goal "Delivers"; answer-key fidelity): no card carries the rules of `origin`, `remoteonly` or `pendingflags`.
   - The Goal names all three among the files whose rules move to the core.
   - No card targets these members: `internal/fabricengine#ReadOrigin`, `#ReadOriginFor`, `#WriteOrigin`, `#MarkBeam`, `#Fabric.RemoteOnlyCommits`, `#markPending`, `#takePending`, `#coalesceFlagged`, `#runHeld`, `#ownedFlagSet`.
   - So these rules get no core row: re-acquire when an owed flag is set again, a corrupt owed flag is taken, a foreign flag stays on disk, and the remote-only tip and commits.
   - The overview's Shared Decisions give no reason for leaving them where they are.

2. **MEDIUM** (D6 criterion "each flow shape has one tier1 seam test that asserts the snapshot its phases read on a real hub"): no card plans one.
   - Card 7 adds the reader `pushJob.read` and card 9 adds `reconcileReader`.
   - Card 8's records-landing facts and card 10's ended facts have readers too.
   - No row asserts the snapshot any of these read on a real hub. The kept tier1 rows assert only end states.
   - The only reader row planned is card 3's `FieldSeams` row in `TestSnapshotReader_ReadsAHub`.

3. **MEDIUM** (D4 criterion "a destructive action whose target turned dirty after the snapshot is refused by destroy.go's checks"): neither planned row meets it as worded.
   - Card 11's new row: the checkout turns dirty inside the verdict hook. That refusal comes from `ffWorktree`'s own dirty check (`internal/fabricengine/originreconcilemoves.go:176-197`), not from `destroy.go`.
   - Card 7's new row is a lease loss on `updateRemoteBranch`, and that request passes `dirtinessNA` (`pushclassified.go:457`), so no dirtiness is checked at execution.
   - Card 7's row also relies on a pre-receive hook landing another clone's push between the read and the leased reset. That hook runs inside the reset push itself, so it would show up as a ref-lock failure, not `gitrepo.ErrLeaseRejected`. The row may not reach the lease branch at `pushclassified.go:465`.

4. **MEDIUM** (D4/D5 criterion "a test per moved verb shows its printed mutations array unchanged"): only `PushPairAnchored` (card 8) and the hub fast-forward reconcile (card 11) get an exact-array row.
   - `CoalescePushBothAt` and `CoalescePushRequest` run the same `pushClassified` through `pushSideDetached`, with no planned array row.
   - The teardown seam's mutations that card 10 appends to the reconcile's record have no array row in fabric. Card 12's moved `TestReconcileOrigin_EndedRuns` is not described as asserting the array.

5. **MEDIUM** (D13 refusal branches): the `OriginCodeTip` refusal for an unsupplied `SeamRunRecordPaths` or `SeamStatusCodec` (card 4) is reached by no row.
   - `TestInstallRequiredSeams` unsets all three seams and asserts only `OriginRunStatus`'s refusal.
   - `TestOriginCodeTip` runs with the seams installed.

6. **MEDIUM** (D13 adapter criterion): the installed run-lock adapter is never checked against the lock shed really takes.
   - Card 11's `TestReconcileOrigin_PrimeLockHeld` replaces `RunLocks` with a path of its own.
   - Card 4's adapter test covers only the status path.
   - So nothing shows that `InstallRequiredSeams`' `PrimeRunLock` is the path shed's prime lock holder uses. Today `TestReconcileOrigin_Moves` holds `shedrun.PrimeRunLock` itself, and card 11 drops that.

7. **MEDIUM** (card 7 branches): these `PushSide`/`JudgePush` branches have no `TestPushSide` row:
   - the branch name cannot be read, so it pushes straight away (`pushclassified.go:132-135`);
   - a Bolt side with nothing unpushed clears its record and returns (`:156-158`);
   - a branch with no upstream proceeds with no origin read (`:749-750`);
   - any other upstream read error is returned (`:753-754`);
   - a non-transport error while judging a gone entry is returned (`:812-815`);
   - `SingleAttempt` returns an origin-lookup transport failure at once (`:820`);
   - the rewind check is skipped for the prime or a pruned ref, and a branch not rewound proceeds (`:772-775`, `:860`).

8. **MEDIUM** (card 7 test cuts): deleted rows pin engine reads that no remaining row holds.
   - `ARejectionOfABranchWithNoUpstreamWhoseNameOriginAlreadyHoldsIsADivergence` and the `RecordsWithNoUpstream` row pin `failRejection`'s fallback to `refs/remotes/origin/<branch>` (`pushclassified.go:672-674`). That is a git read, not a core rule.
   - `ThePrimesCodeBranchPushedAfterAnOriginRewindIsPushedAndRecordsNothing` pins the engine's prime read that skips the rewind check.

9. **MEDIUM** (card 8 branch): `RecordsLandingVerdict` drops today's "any other failed lookup returns its error unclassified" branch (`pushanchored.go:461-462`).
   - The card's verdict list has no such case, and `TestRecordsSideLanding` has no row for it.

10. **MEDIUM** (card 5): the `NoticeTags_*` rows are deleted as "held by the core rows above", but no planned core row covers `FailedPushNotice`.
    - `TestMergeFailedPush` lists no notice row, so the topic, the run (`PairSlug`) and the line go untested.

11. **MEDIUM** (card 6): the engine's `TestDescribePushLockHolder` is deleted as held by `TestCoalesceFlagged_HolderRecordLivesWhileTheLockIsHeld`.
    - That test reaches only the live-holder case.
    - The engine branches left untested: no record file (found false), a read or decode error passed as `readErr`, and skipping the `alive` probe for a record with no pid (`pushlockholder.go:66-79`).

12. **MEDIUM** (card 12 cuts): two deleted rows are claimed as held, but the claimed holders are core rules that cannot see the engine read.
    - The "earlier snapshot tag sets `EarlierSnapshot`" row is deleted as held by `TestEndedTeardowns`. That core row cannot hold the engine's `SnapshotTagged` read (`originreconcileended.go:121`).
    - The `OwnerOf`-error and nil-`OwnerOf` rows are deleted as held by `TestClassifyEntries` "and readOriginOwners' error return in the kept reads". No kept test calls an `OwnerOf` that errors, so the wrapped error (`originreconcile.go:745-748`) and the nil early return (`:730`) go untested.

13. **MEDIUM** (D2 "no verb starts more git processes on any path than its current reads"): card 7 argues this holds without a test.
    - Only the reconcile has call-count rows (the kept moved-nothing and judge-only counts).
    - No row counts the git calls of a push side's phases (pre-lock, under-lock, origin, judgment).

14. **LOW** (D2/D6 fallback to git): cards 7 and 9 read local refs through `gitrepo` in their new readers, outside `snapshotReader`.
    - They do not say whether those readers inherit `readLocalRefs`' fallback for ref storage that is not `files` (`snapshotread.go:162-182`).
    - The fallback test from part 1 covers `snapshotReader` only.

15. **LOW** (card 9 operand): `OriginJudge` refuses an unset `SeamRunLocks` for any scope that is not judge-only.
    - A run scope never takes the prime lock (`originreconcilemoves.go:360`), so a run scope that would not use the seam is refused.
    - D13 says to refuse only "a verb whose run would use one".

16. **LOW** (D5 reach): card 3 installs the seams only when `internal/hubforge` builds a hub.
    - A fabric test binary that builds its hub through `fabricengine.CloneHub` or `fabriccli.CloneAndWire` directly would meet the refusal if it reaches `ReconcileOrigin` before any hubforge call. Such tests exist in, for example, `internal/fabricengine/livestate_verbs_test.go` and `destructivegaps_integration_test.go`.
    - The plan neither argues this away nor adds a row for it.

**Further tool calls**
- Listed hubforge's builders and their callers of `cloneHub`/`copyTemplate`, to confirm every hub-building export reaches card 3's install. My context lacked hubforge's function list.
- Read `templateFor`/`buildTemplate`/`SharedHub` in `internal/hubforge/template.go`, to confirm `SharedHub` builds through `cloneHub` in process. My context lacked that body.
- Grepped the test files that clone directly, without hubforge, for reconcile-like verbs, for finding 16. My context lacked those files' contents.
