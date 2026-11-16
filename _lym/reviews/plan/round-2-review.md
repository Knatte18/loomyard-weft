---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/08-transition-sync-push.md
    summary: The records commit of a shed transition carries no Warp-SHA trailer today, so the code-tip wait, the owed-tip push and the owner guard read nothing
    fix: >-
      In card 8, add `internal/fabricengine#CommitAnchoredPathsWithTrailers` under Uses.
      In its Intent, say that the `Commit` closure of `battenCommitStatusDeps` and of `commitStatusDepsGuardedBy` calls
      `fabricengine.CommitAnchoredPathsWithTrailers(rec, location, paths, msg, nil, opts)` in place of `CommitAnchoredPaths`,
      so every transition, `commitRunRecords`, `commitCancelRecord` and `lym loom commit-records` commit carries `Warp-SHA: <code HEAD>`.
      Add a row to `TestCommitStatusSeam_Real` asserting the records HEAD message carries a `Warp-SHA` trailer equal to the code HEAD.
      In 00-overview.md, reword the `code-tip-is-warp-sha` decision's Rationale from "the trailer does already" to say the two seams write it through the trailer commit.
      Checked with `lym loom validate-plan --overlay`: the added Uses target passes alone and combined.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lym/plan/06-run-hand-over.md, _lym/plan/11-finalize-push-order.md
    summary: No executor in destroy.go resets a local parent branch with reset --keep under an archive-tag ownership, and no card adds one, so archiveParentMerge would run raw git outside destroy.go
    fix: >-
      In card 6, add `plan:internal/fabricengine#resetKeepUnderArchive` -> `func resetKeepUnderArchive(rec *Mutations, req pathRequest, repo *gitrepo.Repo, sha, archiveTag string) error` under Create.
      State in its Intent that it is a new executor in `internal/fabricengine/destroy.go`:
      it runs `checkPathRequest` over a `pathRequest` for the parent worktree (containment, ownership, dirtiness, force, in that order),
      then refuses unless `archiveTag` is a pushed tag on the local tip being left,
      then calls `repo.ResetKeep(sha)` and appends `KindWorktreeReset` to `rec`.
      Make step 3 of `HandOver`'s tips and card 11's `ArchiveParentMerge` call only this executor, and add it to card 11's Uses.
      Name the `pathOwnership` the request declares (the closed set has no archive-tag kind, so either `ownedWarpCheckout` of the parent's repository or one new kind added in this card) and its dirtiness declaration.
      Checked with `lym loom validate-plan --overlay`: the Create in card 6 and the Uses in card 11 pass together with the other proposed fixes.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lym/plan/07-read-run-owed-and-tip.md, _lym/plan/06-run-hand-over.md
    summary: The leased moves of origin's code branch cite an executor by description only, call the lease its ownership, and leave dirtiness unstated although the hand-over runs them over a worktree with uncommitted files
    fix: >-
      In card 7, add `internal/fabricengine#updateRemoteBranch` under Uses and say `resetUnnamedTip` calls it with a `remoteBranchUpdateRequest` over the task pair's warp checkout, `leaseSHA` the origin tip it read.
      Replace "its ownership being the lease on the origin tip it read" in cards 6 and 7 with the real shape:
      ownership is `ownedPairWarpCheckout`, and the lease is the request's `leaseSHA`.
      In card 6 step 1, state the `pathRequest` dirtiness the pushed-tip and reset moves declare;
      the worktree holds the uncommitted files that step 2 archives, so a `dirtyTrackedExcept` declaration would refuse step 1 before the archive.
      The accepted shape is `dirtinessNA` with the reason that the move touches no checkout file.
      Checked with `lym loom validate-plan --overlay`: the added Uses target passes.
  - id: F4
    severity: BLOCKING
    class: scope
    location: _lym/plan/08-transition-sync-push.md
    summary: battencli's hubReadoptSeams CommitFailed calls deps.Spawn() on the commit-status deps whose Spawn field card 8 removes, and no card targets it, so the card gate does not compile
    fix: >-
      In card 8, add `internal/battencli#hubReadoptSeams` under Edit.
      State in the Intent that `CommitFailed` (readopt.go) commits the failed-stop status and pushes through the deps' `Push` in place of `deps.Spawn()`,
      and what an owed push does there (log it and return nil, since the run is already halted at the failed stop; any other push error is returned).
      Update the `CommitFailed` seam doc in `readoptSeams`, which still says it spawns the push.
      Checked with `lym loom validate-plan --overlay`: the added Edit target passes.
  - id: F5
    severity: BLOCKING
    class: scope
    location: _lym/plan/08-transition-sync-push.md, _lym/plan/09-intent-sync-push.md
    summary: loomcli TestPrePushCheckWiredIntoDeps calls statusDeps.Spawn(), the field card 8 removes, but only card 9 edits a prepush test and it edits a different one, so card 8's gate fails to compile the loomcli tests
    fix: >-
      In card 8, add `internal/loomcli#TestPrePushCheckWiredIntoDeps` under Edit and say it drops the `statusDeps.Spawn()` assertion and the "leaves Spawn a nil no-op" wording from its doc.
      Leave card 9's `TestPrePushCheckGuardedSeams` edit as it is, since `guardedSpawn` goes there.
      Checked with `lym loom validate-plan --overlay`: the added Edit target passes.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lym/plan/03-pair-push-records-first.md, _lym/plan/07-read-run-owed-and-tip.md
    summary: A failed-push record is one slot per side and branch that a later failure replaces whole, so a second run's failed push of the shared prime records branch overwrites Run and hides the first run's owed push from ReadRun
    fix: >-
      In card 3, replace `FailedPush.Run string` by `FailedPush.Runs []string` (json `runs,omitempty`) filled from `SyncOptions.RunSlug`,
      and add `internal/fabricengine#recordPushFailure` under Edit so an update unions the existing record's `Runs` with the new run's instead of replacing them.
      In card 7, make `FailedPushesOwedBy` match a record whose `Runs` holds the slug.
      Add to `TestFailedPushRecord` a row where two runs record failures of one prime records branch and both read the record as owed.
      Checked with `lym loom validate-plan --overlay`: the added Edit target passes.
  - id: F7
    severity: LOW
    class: design
    location: _lym/plan/06-run-hand-over.md
    summary: HandOver names no fetch of the prime records branch before droppedIntents compares the local batten record with OriginBattenStatus, and the rejected-push path fetches only the pair's records branch
    fix: >-
      In card 6, state in `HandOver` step 2 that the prime records branch is fetched (through `gitrepo`, under fabric) before `OriginBattenStatus` reads its tip,
      so an intent that already reached origin is not named as dropped and one that did not is not missed.
      Add a `TestPushRun_HandsOverOnATakeover` row where the prime's tracking ref is stale and origin holds the pause intent: the notice names no dropped intent.
---

### [BLOCKING] The records commit names no code tip until the transition seams write the trailer

**Location:** `_lym/plan/08-transition-sync-push.md`, the overview's `code-tip-is-warp-sha` decision.
**Issue:** The decision rests on "the `Warp-SHA` trailer fabric already writes into every records commit".
It does not for the commits that matter.
`battenCommitStatusDeps` (`internal/battencli/commitstatus.go:135`) and `commitStatusDepsGuardedBy` (`internal/loomcli/wiring.go:211`) commit through `fabricengine.CommitAnchoredPaths`.
That calls `CommitRecordsPaths` and `commitRecordsPathsHeld`, which pass the message to `StageAndCommit` untouched (`internal/fabricengine/commitweftpaths.go:41-74`).
Only `CommitAnchoredPathsWithTrailers`, `CommitEmptyWithTrailers` and the fabric commit path (`commitWeftLocked`) append `Warp-SHA`.
No card changes the seams' `Commit`.
Cards 6, 7 and 12 then read nothing from a transition commit:
`lastTransitionCodeTip` finds either no trailer or an older one a fabric commit wrote, `CodeTipMissing` never fires for a real transition, the hand-over pushes no owed tip, and the owner guard resets against a stale tip.
This is D1's "the records commit names the code tip" and every D2 criterion on the code-tip wait failing together.

### [BLOCKING] A local parent branch reset has no executor in destroy.go

**Location:** `_lym/plan/06-run-hand-over.md` (step 3, `archiveParentMerge`), `_lym/plan/11-finalize-push-order.md` (`ArchiveParentMerge`).
**Issue:** Both cards say the `git reset --keep` of the local parent branch "runs through an executor in `internal/fabricengine/destroy.go`" with ownership "the archive tag already pushed".
`PATTERN-fabric-boundary` requires destroying only in `destroy.go`, through an executor that checks containment, ownership, dirtiness and force in that order.
`destroy.go` has no such executor.
Its `reset --keep` sites are `DropLocal` (a pair's own branches), `restoreReplayTip`/`resetKeepOrTag` (a failed replay), `cherryPickOnto` and `resetWeftToIntegrated`, and its path executor `resetHardTo` is `reset --hard`.
The closed `pathOwnershipKind` set has no archive-tag kind.
No card lists `destroy.go` or creates an executor symbol, so an implementer following the cards calls `Repo.ResetKeep` from `archiveParentMerge` directly, which breaks the pattern.
Answer to the focus question for the parent reset: the cards claim the executor and do not plan it.
Round 1's fix for this point (reported as "fixed as proposed") added the claim to the cards' text and no target or symbol, so the gap it was raised for stands.

### [MEDIUM] The leased code-branch moves are described, not named, and their dirtiness is unstated

**Location:** `_lym/plan/07-read-run-owed-and-tip.md` (`resetUnnamedTip`), `_lym/plan/06-run-hand-over.md` (step 1).
**Issue:** `updateRemoteBranch` (`destroy.go:1627`) is the leased-move executor and takes a `remoteBranchUpdateRequest` whose `pathReq` is the pair's warp checkout, ownership `ownedPairWarpCheckout`, with the lease as `leaseSHA`.
Card 6 names it; card 7 does not list it and calls the lease the ownership.
Neither card says which dirtiness the request declares.
The hand-over's step 1 runs before the archive, over a worktree that holds the uncommitted files the archive then commits, so a tracked-change dirtiness refuses step 1 on exactly the case the hand-over exists for.
Answer to the focus question for origin's code branch: the executor exists and fits, the cards must name it and settle the dirtiness.

### [BLOCKING] battencli's readopt seam still calls the removed Spawn

**Location:** `_lym/plan/08-transition-sync-push.md`; code at `internal/battencli/readopt.go:93-99`.
**Issue:** `hubReadoptSeams`' `CommitFailed` builds `battenCommitStatusDeps`, commits the failed-stop status and returns `deps.Spawn()`.
Card 8 turns battencli's `commitStatusDeps` from `Spawn` into `Push` (`commitRunRecords`, `commitCancelRecord`, the "drops its transition-seam row" test edit).
`readopt.go` is in no card's targets, so card 8's gate does not compile.
Which push a failed-stop commit takes is also undecided.

### [BLOCKING] A loomcli test still calls the removed Spawn

**Location:** `_lym/plan/08-transition-sync-push.md`, `_lym/plan/09-intent-sync-push.md`; code at `internal/loomcli/prepush_test.go:409`.
**Issue:** Card 8 removes loom's `commitStatusDeps.Spawn`.
`TestPrePushCheckWiredIntoDeps` calls `statusDeps.Spawn()`.
Card 9 edits `TestPrePushCheckGuardedSeams`, a different test, and it runs after card 8.
The loomcli test package does not compile at card 8's gate.

### [MEDIUM] One failed-push slot per side and branch loses a second run's owed push

**Location:** `_lym/plan/03-pair-push-records-first.md`, `_lym/plan/07-read-run-owed-and-tip.md`; code at `internal/fabricengine/failedpush.go:150-179`.
**Issue:** `recordPushFailure` keys the record by side and branch and writes the new record whole, keeping only `FirstSeen`, `Count` and `Notified`.
Runs of one prime share its records branch.
When run B's push fails after run A's, the record's `Run` becomes B, and `FailedPushesOwedBy` for A returns nothing.
A's transition seam still retries in its own process, but `ReadRun` from a restarted dobby, `lym batten cancel` and `lym beam status` read A as not owed while its transition is not on origin.
Card 7's bound ("an owed push of the prime's records branch holds only the run that wrote it") covers the false hold and leaves this false release open.
A pair's records branch is its own, so only the prime is affected.

### [LOW] The dropped-intent read may use a stale prime tracking ref

**Location:** `_lym/plan/06-run-hand-over.md` (`HandOver` step 2, `OriginBattenStatus`).
**Issue:** `OriginBattenStatus` reads "the prime records branch's fetched upstream tip", as `OriginRunStatus` does, with no network call.
The rejected pair push fetched only the pair's records branch (`retryWeftPush`).
The notice then compares the local batten record with whatever the last fetch left, so it can name an intent already on origin or miss one that is not.
The hand-over's archive and push are unaffected; only the notice text is.

## Focus verdicts

- Nil return without landing: not a defect.
  Card 3 returns an error for a deferred or ended records side through `HasUnpushed`, and card 4's `landed` does the same in `coalescePushSides`, so the seam cannot read either as landed.
- Branch moves backwards through an executor: see F2 and F3.
- Fields and signatures removed or changed: `PushPairAnchored`, `CoalescePushRequest`, `DetachedPush`, `PushOwedOffline` and `retryWeftPush` have every caller and test in their cards' targets.
  The gaps are the `Spawn` removal (F4, F5).
- Origin reads from the right branch: `ReadUpstreamTakeover`, `OriginCodeTip` and `OriginBattenStatus` read the child record from the pair's branch and the batten record from the prime's, as D2 places them.
  The trailer they rely on is not written (F1), and the batten read may be stale (F7).
- Force-moves and give-ups: card 7's Intent states what the guard drops and its three bounds, and cards 6 and 12 push a tip this machine holds before it is given up.
  Not a defect beyond F3.
