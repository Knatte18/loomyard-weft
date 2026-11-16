# Card 6 — run-hand-over

**Create:**
- `plan:internal/fabricengine#HandOverRequest` -> `type HandOverRequest struct`
- `plan:internal/fabricengine#HandOverResult` -> `type HandOverResult struct`
- `plan:internal/fabricengine#HandOverTips` -> `func HandOverTips(prime *lymcwd.Location, req HandOverRequest) (HandOverResult, error)`
- `plan:internal/fabricengine#WriteHandOverMark` -> `func WriteHandOverMark(hubPath, slug string, mark HandOverMark) error`
- `plan:internal/fabricengine#OriginBattenStatus` -> `func OriginBattenStatus(prime *lymcwd.Location, slug string) (status shedengine.Status, found bool, err error)`
- `plan:internal/fabricengine#archiveParentMerge` -> `func archiveParentMerge(rec *Mutations, parentDir, slug string) (tag string, err error)`
- `plan:internal/fabricengine#resetKeepUnderArchive` -> `func resetKeepUnderArchive(rec *Mutations, req pathRequest, repo *gitrepo.Repo, sha, archiveTag string) error`
- `plan:internal/fabricengine#lastTransitionCodeTip` -> `func lastTransitionCodeTip(weftPath, ref, slug string) (tip string, err error)`
- `plan:internal/fabricengine#pushOwedCodeTip` -> `func pushOwedCodeTip(rec *Mutations, pair *lymcwd.Location) (pushed bool, err error)`
- `plan:internal/fabricengine#owedTipPushedHook` -> `var owedTipPushedHook func()`
- `plan:internal/fabricengine#SetOwedTipPushedHookForTest` -> `func SetOwedTipPushedHookForTest(t testing.TB, hook func())`
- `plan:internal/fabricengine#TransitionTrailerKey` -> `const TransitionTrailerKey = "Run-Transition"`
- `plan:internal/fabricengine_test#TestHandOverTips` -> `func TestHandOverTips(t *testing.T)`
- `plan:internal/pairteardown#ThisMachine` -> `func ThisMachine() (fabricengine.MachineNames, error)`
- `plan:internal/pairteardown#RunPush` -> `type RunPush struct`
- `plan:internal/pairteardown#PushRun` -> `func PushRun(prime, l *lymcwd.Location, slug string, push RunPush) (fabricengine.PushResult, error)`
- `plan:internal/pairteardown#HandOver` -> `func HandOver(prime *lymcwd.Location, slug string, entry shedengine.Takeover, push RunPush) error`
- `plan:internal/pairteardown#droppedIntents` -> `func droppedIntents(local, origin shedengine.Status) []string`
- `plan:internal/pairteardown_test#TestPushRun_HandsOverOnATakeover` -> `func TestPushRun_HandsOverOnATakeover(t *testing.T)`

**Edit:**
- `internal/fabricengine#OwnerArchivePair`
- `internal/fabricengine#archiveOwnerPair`
- `internal/fabricengine#archiveOwnerTip`
- `internal/fabricengine#KindWorktreeReset`
- `internal/fabricengine#updateRemoteBranch`
- `internal/fabricengine/doc.go`
- `internal/pairteardown/doc.go`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes`
- `internal/fabriccli#addWeftVerbs`

**Uses:**
- `internal/pairteardown#EndSubjectProcesses`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#ReadBattenRecord`
- `internal/fabricengine#ArchiveOwnerTips`
- `internal/fabricengine#ReadUpstreamTakeover`
- `internal/fabricengine#ReadHandOverMark`
- `internal/testkit/takeoverkit#WriteTakeover`
- `internal/loomcli#primeOfHub`
- `internal/loomcli/doc.go`
- `internal/fabriccli/doc.go`

**Intent:**
D2's hand-over, run when a run's records push meets another machine's takeover entry.
`internal/fabricengine` does the git steps, since fabric owns every git op on the pair (`PATTERN-fabric-boundary`);
`internal/pairteardown` ends the processes and removes the pair, since it owns teardown and imports fabricengine, and never the reverse.

`HandOverTips` takes `HandOverRequest`: `Slug`, `Pair` (the task pair, nil for a prime-only run), `Parent` (the parent pair the run's parent-side merge went into, nil when none), `Entry` (the takeover entry) and `DroppedIntents` (the pause or cancel intents to name).
Under the slug's teardown lock, in order:
1. The owed code tip: `pushOwedCodeTip` reads, from the records worktree's fetched upstream, the code tip the run's last transition on origin names (`lastTransitionCodeTip`).
   When origin's code branch lacks it and this repository holds it, it pushes that tip with a lease on origin's code branch at the value it read with the records.
   `owedTipPushedHook`, nil in production, runs right after that push, so a test can give the tip up on origin between the push and the re-read.
   `SetOwedTipPushedHookForTest`, in `internal/fabricengine/export_test.go`, sets it and restores nil in `t.Cleanup`, as `SetArchiveTagPushHookForTest` does;
   `TestHandOverTips`' give-up row calls it, since the external test package cannot set the unexported hook.
   After the push it fetches the records branch again; when the last transition no longer names the tip, it resets origin's code branch with a lease to the value it read, so a given-up tip does not stay (D2).
2. The archive: `ArchiveOwnerTips` over the pair, with `OwnerArchivePair.CommitUncommitted` set, so `archiveOwnerTip` first commits the worktree's uncommitted files onto a commit made off the local tip, never onto the branch, and tags that commit.
3. The local parent branch: when `Parent` is set and the run's parent-side merge alone puts the parent branch ahead of its upstream, `archiveParentMerge` tags the local tip `archive-owner/<slug>/<sha12>`, pushes the tag, and resets the local parent branch to its upstream with `git reset --keep`.
4. The tags are pushed (steps 2 and 3 push each), and then one notice goes through the installed push notifier: the slug, the new owner machine, and each dropped intent with the way forward of repeating it on that machine.
5. `WriteHandOverMark` writes the mark naming the entry's machine, after the tags landed.
Step 1's two leased moves of origin's code branch, the push and the reset, run through `updateRemoteBranch`, `destroy.go`'s executor of the leased move, never `Repo.UpdateRemoteBranchLeased` directly.
Each passes a `remoteBranchUpdateRequest` whose `pathReq` is the task pair's warp checkout with ownership `ownedPairWarpCheckout`, and whose `leaseSHA` is the origin tip it read.
That `pathReq` declares `dirtinessNA`, the reason being that a move of origin's branch touches no checkout file:
the worktree still holds the uncommitted files step 2 archives, so a tracked-change declaration would refuse step 1 before the archive.
`updateRemoteBranch` wraps a lost lease's error with `gitrepo.ErrLeaseRejected`, its divergence text unchanged, so a caller tells a lost lease from a failed move.
A lost lease in step 1 returns an error naming that origin's code branch moved during the hand-over, whose way forward is that the next rejected push hands over again;
the reset's divergence advice is not passed on, since this run ran no reset.
Step 3's `git reset --keep` runs only through `resetKeepUnderArchive`, a new executor in `internal/fabricengine/destroy.go`, never `Repo.ResetKeep` directly.
It runs `checkPathRequest` over a `pathRequest` for the parent worktree, contained in the hub, which checks containment, ownership, dirtiness and force in that order (`PATTERN-fabric-boundary`).
Its ownership is `ownedWarpCheckout` of the parent's warp repository, which counts any worktree of that repository, the prime checkout included.
Its dirtiness is `dirtinessNA`, the reason being that `git reset --keep` carries uncommitted changes across and refuses by itself where one touches a file the reset moves.
It then refuses unless `archiveTag` is on origin at the local tip being left, calls `repo.ResetKeep(sha)`, and appends `KindWorktreeReset` to rec;
`KindWorktreeReset`'s doc describes a worktree reset of either kind, hard or keep, naming no executor.
`HandOverTips` pushes nothing to any branch but step 1's leased tip.
A failure at any step returns it, wrapping `ErrArchiveFailed` for steps 2 and 3, before the mark is written, so the next rejected push hands over again.
`HandOverResult` lists the tags and whether a tip was pushed or reset.

`TransitionTrailerKey` sits in `internal/fabricengine/trailer.go` beside `WarpSHATrailerKey` (overview, `code-tip-is-warp-sha`):
the transition seams (card 8) and the give-up (card 12) write it, and no other fabric commit does.
`lastTransitionCodeTip` returns the `Warp-SHA` of the newest commit on ref whose `TransitionTrailerKey` value is slug,
skipping fabric's other weft commits, which carry no `Run-Transition`, and any other run's transition.

`RunPush` holds `LockWait`, `TeardownWait`, `InsidePair` (the caller runs inside the task pair) and `Parent` (the parent pair a Finalize merged into, nil otherwise; card 11 sets it).

`pairteardown.HandOver` runs D2's sequence over it:
1. `EndSubjectProcesses(prime, slug, false)`; the caller and its ancestors are never ended.
   The dobby end stays unset: every caller in this part is a run its parent's dobby drives (`PATTERN-dobby-runs`), never a dobby, so the dobby that hands the subject over is left to do so.
2. The dropped intents: `droppedIntents` names the pause intent and the unfinished cancel intent the local batten record holds (`ReadBattenRecord`) that origin's fetched batten record lacks (`OriginBattenStatus`).
   `OriginBattenStatus` first fetches the prime records branch from origin, through `gitrepo` under fabric, then reads slug's batten status file at its fetched upstream tip.
   So the read does not depend on whether, or when, an earlier fetch refreshed the prime records branch: an intent already on origin is not named dropped, and one not on origin is not missed.
3. `HandOverTips` with the pair, `push.Parent` and those intents.
4. Unless `push.InsidePair`, the local removal through `Teardown.RemovePair` with `Request{Slug, Local: true, Force: true, Verdict: fabricengine.EntryOwnedElsewhere}`, the uncommitted files being under the archive tag by then.
   Inside the pair the removal is left to the parent's dobby, and the returned error says the pair is left standing for it.
It returns an error wrapping `fabricengine.ErrHandedOver` naming the new owner, so its caller stops; a failed step returns that step's error instead, naming the slug and the step.

`PushRun` is the run's synchronous pair push: it fills `PairPush` from `RunPush` (`LockWait`, `TeardownWait`, `RunSlug: slug`) and `ThisMachine`'s names, calls `PushPairAnchored` on l, and on a `*TakeoverError` runs `HandOver` with its entry.
`ErrHandedOver` from the push passes through unchanged: a mark already names the entry's machine, so nothing more is pushed.
`ThisMachine` builds `MachineNames` from `machinename.Names`; the registry part adds its names here.
`addWeftVerbs` builds its `MachineNames` through `pairteardown.ThisMachine` in both branches, replacing card 5's `machinename.Names` build (fabriccli already imports pairteardown).

The first production caller is loom's `Push` seam behind `lym loom commit-records`: `commitStatusDepsGuardedBy` calls `PushRun(prime, location, runID, RunPush{LockWait: StatusPushLockWait, InsidePair: true})`, prime from `primeOfHub`.
`commitRecordsVerb` reports an `ErrHandedOver` error as an error envelope naming the new owner, whose way forward is to drive the run on that machine.
So after a takeover, `lym loom commit-records` puts no code commit on origin's code branch (D2's criterion).
The pairteardown doc gains a "Handing a run over" section; the fabric doc gains `HandOverTips` and the mark's writer.

Tests:
- `TestHandOverTips` (tier1) over a pair fixture with a takeover planted on origin. Rows:
  local commits and an uncommitted file end under a pushed `archive-owner/` tag and origin's branches are unchanged;
  a parent branch ahead by this run's merge alone is tagged and reset level with its upstream;
  a tip the last transition names is pushed under the lease;
  a tip whose transition the test gives up on origin through `owedTipPushedHook`, set by `SetOwedTipPushedHookForTest`, is reset off origin;
  the mark exists only after the tags are on origin, and a tag push refused by origin leaves no mark and returns the failure;
  a request carrying a dropped pause intent posts one notice naming the machine and the intent, through a recording notifier.
- `TestPushRun_HandsOverOnATakeover` (tier1, hubforge) over a hub with a registered sleeping process under the slug:
  outside the pair the process is ended, the pair is gone and the mark stands; inside it the pair stays and the error names the pair left for the parent;
  a local batten record holding a pause intent origin's lacks names it in the notice;
  a stale prime tracking ref, with origin holding the pause intent the local record holds, names no dropped intent in the notice;
  a second `PushRun` returns `ErrHandedOver` and pushes nothing.
  A former name is card 5's row and is not repeated here.
- `TestCommitRecordsVerb_ErrorEnvelopes` (tier0) gains the hand-over row.

**ImpactSummary:** Adds the hand-over in fabricengine and pairteardown and moves one loomcli seam onto it; `OwnerArchivePair` gains an opt-in field, so the existing owner-archive callers keep their behaviour.
