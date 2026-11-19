# Card 13 — records-only-replay

**Create:**
- `plan:internal/fabricengine#cherryPickReworded` -> `func cherryPickReworded(rec *Mutations, req pathRequest, repo *gitrepo.Repo, onto string, commits []string, reword func(message string) string) (dropped []string, err error)`
- `plan:internal/fabricengine#unpushedCodeCommits` -> `func unpushedCodeCommits(worktree, upstream string) ([]string, error)`
- `plan:internal/fabricengine#rewordWarpSHA` -> `func rewordWarpSHA(replayed map[string]bool, forkPoint string) func(message string) string`
- `plan:internal/fabricengine_test#TestReplay_OneRepository` -> `func TestReplay_OneRepository(t *testing.T)`

**Edit:**
- `internal/fabricengine#replayWeft`
- `internal/fabricengine#Fabric.Pull`
- `internal/fabricengine#Fabric.settleUnmovedWeft`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#reconcileRecordsBranch`
- `internal/fabricengine#Fabric.FastForwardPair`
- `internal/fabricengine#Fabric.syncSideBeforeMerge`
- `internal/fabricengine#integrateWeft`
- `internal/fabricengine#recordPushFailure`
- `internal/fabricengine#FailedPushWayForward`
- `internal/fabricengine_test#TestFailedPushRecord`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#cherryPickOnto`
- `internal/fabricengine#IsRecordsOnlyCommit`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#parseWarpSHATrailer`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#AddPairWith`

**Intent:** D7's replay in a one-repository hub, on any code branch: only fabric's own unpushed records-only commits are rewritten, never a code commit. `lym-bolt` keeps today's by-entry Bolt replay.

- `unpushedCodeCommits` lists the local commits not reachable from `upstream` that `IsRecordsOnlyCommit` reads false, oldest first.
- `replayWeft` in a one-repository worktree: when `unpushedCodeCommits` is non-empty it rewrites nothing and returns a stop naming them.
  Otherwise it replays the records-only commits through `cherryPickReworded`, sets aside ended-run commits only among those as today,
  and rewords each replayed commit's `Warp-SHA` trailer through `rewordWarpSHA`: a trailer naming a commit being replayed becomes the fork point, `git merge-base HEAD upstream`, whose code is the code the commit was made over; any other `Warp-SHA` stays, so no trailer names code origin added.
- `cherryPickReworded` is `cherryPickOnto` with each pick committed under `reword(message)`; it lives in `destroy.go` beside it with the same dirtiness check, abort and tip restore. `replayBolt` keeps `cherryPickOnto`.
- The stop is owed and notified, naming the commits: `retryWeftPush` records it as a deferred push with `Commits` set, after it has read the takeover as today, so a rejected push still hands over on another machine's takeover.
  `recordPushFailure` sends a deferred record's notice when it carries `Commits`, as it sends a refused push's, so the orch hears of the wait;
  `FailedPushWayForward` of such a record names the commits and the settling step, `git merge origin/<branch>` in the worktree, settling any conflict and committing, then `lym fabric push` in its anchored directory.
  `settleUnmovedWeft` returns it as a `*WeftStopError` whose way forward names the same step; `reconcileRecordsBranch` reports the branch diverged.
- `Fabric.Pull` skips its warp arm in a one-repository worktree, since the records arm moves the one branch.
- Bringing a branch level with its own origin tip: `Fabric.FastForwardPair` and `Fabric.syncSideBeforeMerge`, which `Fabric.Merge` runs on the receiving branch, in a one-repository worktree commit the worktree's records first (D4), judge code paths only for dirtiness,
  then replay the local records-only commits onto origin's tip as above, and refuse only when a local non-records commit is unpushed (`FastForwardNotFastForward`, and the not-synced guard reason for the merge).
- The anchor change's integrate kind (`integrateWeft`), chosen when origin's tip carries another anchor, in a one-repository worktree replays the local records-only commits onto origin's tip as above,
  each pick's records paths moved from the local anchor's `_lym` to the one origin's marker names before it is committed; a conflict stops as a replay conflict does, and the stop's way forward is unchanged.

`TestReplay_OneRepository` (tier1, `NewOneRepoHub`, a pair, and a `CloneSecond` hub pushing to the same branch) rows:
- `RecordsOnlyCommitsReplayWithTrailersRewrittenToTheForkPoint`;
- `CodeCommitAmongOriginsNewCommitsStillReplaysAndNoTrailerNamesIt`;
- `EndedRunCommitsAreSetAsideAmongTheRecordsOnlyCommits`;
- `LocalCodeCommitRewritesNothingAndStopsNamingIt` (pull returns the stop naming the commit and the merge-then-push way forward; the branch is unmoved);
- `RejectedPushWithALocalCodeCommitIsOwedAndNotified` (the push records a deferred failure naming the commit, read owed by `IsPushOwed`, with one notice through a recording notifier);
- `RejectedPushReadsTheTakeoverFirst` (the second hub's takeover entry on origin: the push hands over before any stop);
- `ReconcileReportsTheBranchDiverged` (origin reconcile over a local code commit and a diverged origin);
- `CodeCommitCarryingTheMergeTrailerIsNotReplayed`;
- `FastForwardPairLevelsWithOriginKeepingRecordsCommits`;
- `FastForwardPairRefusesOverALocalCodeCommit`;
- `MergeIntoAParentBehindOriginWithRecordsCommitsSyncs` (a parent with a local records-only commit and origin moved on: `Fabric.Merge` of a task lands);
- `OriginAtAnotherAnchorIntegrates` (a second clone moves the anchor and pushes; `lym fabric anchor --set` in the first pair carries its local records-only commit onto origin's tip at the new anchor).
`TestFailedPushRecord` (tier0) gains rows asserting a deferred record with `Commits` is notified once and its way forward names the commits and `git merge origin/<branch>`.

**ImpactSummary:** Replay, pull, the records push retry, origin reconcile, FastForwardPair, the merge's sync, the anchor integrate and deferred-record notices gain one-repository branches; replayBolt and two-repository replay are unchanged.
