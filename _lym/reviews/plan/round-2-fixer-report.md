# Round 2 fixer-report — plan

Every finding's premise holds against the code at HEAD; all seven are fixed.
`lym loom validate-plan` passes after the round's edits and after the self-check's.

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 8 lists `internal/fabricengine#CommitAnchoredPathsWithTrailers` under Uses.
  Its seams section opens with a commit bullet: both seams' `Commit` closures call `CommitAnchoredPathsWithTrailers(rec, location, paths, msg, nil, opts)` in place of `CommitAnchoredPaths`, so every records commit either seam makes carries `Warp-SHA: <code HEAD>`.
  `TestCommitStatusSeam_Real` gains the row asserting the records HEAD's `Warp-SHA` equals the code HEAD.
  The overview's `code-tip-is-warp-sha` decision drops "fabric already writes into every records commit", names the trailer commit both seams write through, rewords the Rationale as proposed, and adds card 8 to Applies to.
  Evidence: `commitweftpaths.go:41-82` writes no trailer; `trailercommit.go:78-116` writes `Warp-SHA`.
- **F2** (BLOCKING), fixed as proposed.
  Card 6 creates `resetKeepUnderArchive` with the proposed signature and states it in the Intent as a new `destroy.go` executor:
  `checkPathRequest` over the parent worktree, contained in the hub, ownership `ownedWarpCheckout` of the parent's warp repository (no new kind), dirtiness `dirtinessNA` because `git reset --keep` carries uncommitted changes across and refuses by itself on a touched file;
  then the archive-tag-on-origin refusal, `repo.ResetKeep(sha)` and `KindWorktreeReset`.
  Step 3 and card 11's `ArchiveParentMerge` reset only through it; card 11 lists it under Uses.
  Card 6 also edits `KindWorktreeReset`'s doc, whose text names only `resetHardTo` today (`mutation.go:43`).
- **F3** (MEDIUM), fixed as proposed.
  Card 7 lists `internal/fabricengine#updateRemoteBranch` and states `resetUnnamedTip`'s request: ownership `ownedPairWarpCheckout`, `leaseSHA` the origin tip it read, dirtiness `dirtinessNA`.
  Cards 6 and 7 no longer call the lease the ownership.
  Card 6 step 1 declares `dirtinessNA` with the reason that the move touches no checkout file, since step 2's uncommitted files would make a tracked-change declaration refuse.
- **F4** (BLOCKING), fixed as proposed.
  Card 8 edits `internal/battencli#hubReadoptSeams` and `internal/battencli#readoptSeams`.
  `CommitFailed` pushes through the deps' `Push`; an owed push is logged and returns nil, any other push error is returned; the `readoptSeams` doc says it pushes.
  The ImpactSummary names batten's failed-stop commit.
- **F5** (BLOCKING), fixed as proposed.
  Card 8 edits `internal/loomcli#TestPrePushCheckWiredIntoDeps`: it drops the `statusDeps.Spawn()` assertion and the "leaves Spawn a nil no-op" doc wording. Card 9's edit is unchanged.
- **F6** (MEDIUM), fixed as proposed, with the test row split between the two cards that own the behaviours.
  Card 3: `FailedPush.Runs []string` (json `runs,omitempty`); `recordPushFailure` and `RecordPushFailure` (whose doc says later failures replace the other fields, `failedpush.go:136-138`) are Edit targets, and an update unions `Runs`.
  Card 3's `TestFailedPushRecord` gains the union row (the record holds both runs); card 7's gains the row where each of the two runs reads it owed, since `FailedPushesOwedBy` is card 7's.
  Card 7's `FailedPushesOwedBy` matches a record whose `Runs` holds the slug; the overview's `owed-is-a-failed-push-record` decision reads `FailedPush.Runs`.
- **F7** (LOW), fixed as proposed.
  Card 6's `HandOver` step 2: `OriginBattenStatus` first fetches the prime records branch from origin, through `gitrepo` under fabric, then reads.
  `TestPushRun_HandsOverOnATakeover` gains the stale-tracking-ref row naming no dropped intent.

## Self-check

- fix-consistency, LOW, card 3's ImpactSummary omits the `Runs` union: fixed as proposed (the summary names it).
- fix-consistency, LOW, card 6's `KindWorktreeReset` doc sentence lists executors and misses `destroy.go:2277`'s reset: fixed as proposed (the doc describes a hard or keep worktree reset, naming no executor).
- fix-claims, MEDIUM, card 6 step 2's claim that the rejected pair push fetched only the pair's records branch is unsupported (`gitrepo.Fetch` runs a bare `git fetch`): fixed as proposed (the reason now rests on not depending on any earlier fetch; the fetch itself stays).
- fix-proposal, MEDIUM, `updateRemoteBranch` does not wrap `ErrLeaseRejected` and returns the reset's divergence advice (`destroy.go:1641-1646`): fixed as proposed.
  Card 6 moves `updateRemoteBranch` from Uses to Edit: it wraps a lost lease with `gitrepo.ErrLeaseRejected`, keeping its text.
  Card 6 step 1 reports a lost lease as origin's code branch moving during the hand-over, retried by the next rejected push.
  Card 7's guard keeps the rejection on a lost lease and returns any other failure.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
