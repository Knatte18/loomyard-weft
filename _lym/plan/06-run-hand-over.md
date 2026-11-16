# Card 6 — run-hand-over

**Create:**
- `plan:internal/fabricengine#HandOverRequest` -> `type HandOverRequest struct`
- `plan:internal/fabricengine#HandOverResult` -> `type HandOverResult struct`
- `plan:internal/fabricengine#HandOverTips` -> `func HandOverTips(prime *lymcwd.Location, req HandOverRequest) (HandOverResult, error)`
- `plan:internal/fabricengine#WriteHandOverMark` -> `func WriteHandOverMark(hubPath, slug string, mark HandOverMark) error`
- `plan:internal/fabricengine#archiveParentMerge` -> `func archiveParentMerge(rec *Mutations, parentDir, slug string) (tag string, err error)`
- `plan:internal/fabricengine#lastTransitionCodeTip` -> `func lastTransitionCodeTip(weftPath, ref string) (tip string, err error)`
- `plan:internal/fabricengine#pushOwedCodeTip` -> `func pushOwedCodeTip(rec *Mutations, pair *lymcwd.Location) (pushed bool, err error)`
- `plan:internal/fabricengine#owedTipPushedHook` -> `var owedTipPushedHook func()`
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
- `internal/fabricengine/doc.go`
- `internal/pairteardown/doc.go`
- `internal/loomcli#commitStatusDepsGuardedBy`
- `internal/loomcli#commitRecordsVerb`
- `internal/loomcli#TestCommitRecordsVerb_ErrorEnvelopes`

**Uses:**
- `internal/pairteardown#EndSubjectProcesses`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#ReadBattenRecord`
- `internal/fabricengine#ArchiveOwnerTips`
- `plan:internal/fabricengine#ReadUpstreamTakeover`
- `plan:internal/fabricengine#ReadHandOverMark`
- `internal/fabricengine#OriginRunStatus`
- `internal/gitrepo#Repo.UpdateRemoteBranchLeased`
- `plan:internal/testkit/takeoverkit#WriteTakeover`
- `internal/loomcli#primeOfHub`
- `internal/loomcli/doc.go`

**Intent:**
D2's hand-over, run when a run's records push meets another machine's takeover entry.
`internal/fabricengine` does the git steps, since fabric owns every git op on the pair (`PATTERN-fabric-boundary`);
`internal/pairteardown` ends the processes and removes the pair, since it owns teardown and imports fabricengine, and never the reverse.

`HandOverTips` takes `HandOverRequest`: `Slug`, `Pair` (the task pair, nil for a prime-only run), `Parent` (the parent pair the run's parent-side merge went into, nil when none), `Entry` (the takeover entry) and `DroppedIntents` (the pause or cancel intents to name).
Under the slug's teardown lock, in order:
1. The owed code tip: `pushOwedCodeTip` reads, from the records worktree's fetched upstream, the code tip the run's last transition on origin names (`lastTransitionCodeTip`, the newest commit's `Warp-SHA` trailer on that ref).
   When origin's code branch lacks it and this repository holds it, it pushes that tip with a lease on origin's code branch at the value it read with the records (`Repo.UpdateRemoteBranchLeased`).
   `owedTipPushedHook`, nil in production, runs right after that push, so a test can give the tip up on origin between the push and the re-read.
   After the push it fetches the records branch again; when the last transition no longer names the tip, it resets origin's code branch with a lease to the value it read, so a given-up tip does not stay (D2).
2. The archive: `ArchiveOwnerTips` over the pair, with `OwnerArchivePair.CommitUncommitted` set, so `archiveOwnerTip` first commits the worktree's uncommitted files onto a commit made off the local tip, never onto the branch, and tags that commit.
3. The local parent branch: when `Parent` is set and the run's parent-side merge alone puts the parent branch ahead of its upstream, `archiveParentMerge` tags the local tip `archive-owner/<slug>/<sha12>`, pushes the tag, and resets the local parent branch to its upstream with `git reset --keep`.
4. The tags are pushed (steps 2 and 3 push each), and then one notice goes through the installed push notifier: the slug, the new owner machine, and each dropped intent with the way forward of repeating it on that machine.
5. `WriteHandOverMark` writes the mark naming the entry's machine, after the tags landed.
It pushes nothing to any branch but step 1's leased tip.
A failure at any step returns it, wrapping `ErrArchiveFailed` for steps 2 and 3, before the mark is written, so the next rejected push hands over again.
`HandOverResult` lists the tags and whether a tip was pushed or reset.

`RunPush` holds `LockWait`, `TeardownWait`, `InsidePair` (the caller runs inside the task pair) and `Parent` (the parent pair a Finalize merged into, nil otherwise; card 11 sets it).

`pairteardown.HandOver` runs D2's sequence over it:
1. `EndSubjectProcesses(prime, slug, false)`; the caller and its ancestors are never ended.
   The dobby end stays unset: every caller in this part is a run its parent's dobby drives (`PATTERN-dobby-runs`), never a dobby, so the dobby that hands the subject over is left to do so.
2. The dropped intents: `droppedIntents` names the pause intent and the unfinished cancel intent the local batten record holds (`ReadBattenRecord`) that origin's fetched batten record lacks (`OriginRunStatus` over the prime's records branch).
3. `HandOverTips` with the pair, `push.Parent` and those intents.
4. Unless `push.InsidePair`, the local removal through `Teardown.RemovePair` with `Request{Slug, Local: true, Force: true, Verdict: fabricengine.EntryOwnedElsewhere}`, the uncommitted files being under the archive tag by then.
   Inside the pair the removal is left to the parent's dobby, and the returned error says the pair is left standing for it.
It returns an error wrapping `fabricengine.ErrHandedOver` naming the new owner, so its caller stops; a failed step returns that step's error instead, naming the slug and the step.

`PushRun` is the run's synchronous pair push: it fills `PairPush` from `RunPush` (`LockWait`, `TeardownWait`, `RunSlug: slug`) and `ThisMachine`'s names, calls `PushPairAnchored` on l, and on a `*TakeoverError` runs `HandOver` with its entry.
`ErrHandedOver` from the push passes through unchanged: a mark already names the entry's machine, so nothing more is pushed.
`ThisMachine` builds `MachineNames` from `machinename.Names`; the registry part adds its names here.

The first production caller is loom's `Push` seam behind `lym loom commit-records`: `commitStatusDepsGuardedBy` calls `PushRun(prime, location, runID, RunPush{LockWait: StatusPushLockWait, InsidePair: true})`, prime from `primeOfHub`.
`commitRecordsVerb` reports an `ErrHandedOver` error as an error envelope naming the new owner, whose way forward is to drive the run on that machine.
So after a takeover, `lym loom commit-records` puts no code commit on origin's code branch (D2's criterion).
The pairteardown doc gains a "Handing a run over" section; the fabric doc gains `HandOverTips` and the mark's writer.

Tests:
- `TestHandOverTips` (tier1) over a pair fixture with a takeover planted on origin. Rows:
  local commits and an uncommitted file end under a pushed `archive-owner/` tag and origin's branches are unchanged;
  a parent branch ahead by this run's merge alone is tagged and reset level with its upstream;
  a tip the last transition names is pushed under the lease;
  a tip whose transition the test gives up on origin through `owedTipPushedHook` is reset off origin;
  the mark exists only after the tags are on origin, and a tag push refused by origin leaves no mark and returns the failure;
  a request carrying a dropped pause intent posts one notice naming the machine and the intent, through a recording notifier.
- `TestPushRun_HandsOverOnATakeover` (tier1, hubforge) over a hub with a registered sleeping process under the slug:
  outside the pair the process is ended, the pair is gone and the mark stands; inside it the pair stays and the error names the pair left for the parent;
  a local batten record holding a pause intent origin's lacks names it in the notice;
  a second `PushRun` returns `ErrHandedOver` and pushes nothing.
  A former name is card 5's row and is not repeated here.
- `TestCommitRecordsVerb_ErrorEnvelopes` (tier0) gains the hand-over row.

**ImpactSummary:** Adds the hand-over in fabricengine and pairteardown and moves one loomcli seam onto it; `OwnerArchivePair` gains an opt-in field, so the existing owner-archive callers keep their behaviour.
