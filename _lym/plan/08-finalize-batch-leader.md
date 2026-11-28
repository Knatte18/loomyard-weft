# Card 8 — finalize-batch-leader

**Create:**
- `plan:internal/landingshed#DetachDeps` -> `type DetachDeps struct`
- `plan:internal/landingshed#batchMember` -> `type batchMember struct`
- `plan:internal/landingshed#Finalize.takeBatch` -> `func (fz *Finalize) takeBatch(ctx context.Context) ([]batchMember, error)`
- `plan:internal/landingshed#Finalize.leadBatch` -> `func (fz *Finalize) leadBatch(ctx context.Context, parent parentMerger, members []batchMember, notice string) (bool, *ending)`
- `plan:internal/landingshed#Finalize.gateChainCommit` -> `func (fz *Finalize) gateChainCommit(ctx context.Context, commit string) (verifytree.Result, string, *ending)`
- `plan:internal/landingshed#Finalize.judgeMember` -> `func (fz *Finalize) judgeMember(ctx context.Context, member batchMember) (bool, error)`
- `plan:internal/landingshed#Finalize.landPrefix` -> `func (fz *Finalize) landPrefix(ctx context.Context, parent parentMerger, base string, prefix []fabricengine.ChainCommit, members []batchMember, notice string) (bool, *ending)`
- `plan:internal/landingshed#newBatchGate` -> `func newBatchGate(deps Deps) verifyGate`
- `plan:internal/landingshed#verifyGate.checkResult` -> `func (g verifyGate) checkResult(ctx context.Context, producer, parentBranch string) (string, bool, verifytree.Result, error)`
- `plan:internal/landingshed#recordingParentMerger.MoveCodeTo` -> `func (m *recordingParentMerger) MoveCodeTo(base, tip string) (bool, error)`
- `plan:internal/landingshed_test#TestFinalizeQueue_LandsABatch` -> `func TestFinalizeQueue_LandsABatch(t *testing.T)`
- `plan:internal/landingshed_test#TestFinalizeQueue_LeavesOutAMember` -> `func TestFinalizeQueue_LeavesOutAMember(t *testing.T)`
- `plan:internal/loomcli#memberLandingDeps` -> `func memberLandingDeps(l *lymcwd.Location, cfg landingshed.Config, registry modelspec.Registry, runner *shuttleengine.Runner) func(pairPath string) (landingshed.Deps, error)`
- `plan:internal/loomcli#judgeOnlyReconciler` -> `func judgeOnlyReconciler(l *lymcwd.Location, slug string) func() (fabriclayout.OriginReport, error)`
- `plan:internal/loomcli#TestLandingDeps_TaskHeadReadsTheBranchWhileDetached` -> `func TestLandingDeps_TaskHeadReadsTheBranchWhileDetached(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#parentMerger`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed#verifyGate.check`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Uses:**
- `internal/landingqueue#Store`
- `internal/fabricengine#Fabric.BuildLandingChain`
- `internal/fabricengine#Fabric.MoveCodeTo`
- `internal/fabricengine#Fabric.CheckoutDetached`
- `internal/fabricengine#Fabric.CheckoutBranch`
- `internal/fabricengine#WriteDetachMarker`
- `internal/fabricengine#RemoveDetachMarker`
- `internal/fabricengine#MergeGuardError`
- `internal/landingshed#Config.BatchSize`
- `internal/landingshed#Finalize.actOnOutcome`
- `internal/landingshed#Finalize.recordLanding`
- `internal/landingshed#Finalize.ownerChecks`
- `internal/landingshed#Finalize.landCheck`
- `internal/landingshed#Finalize.needsGitHubOnDisk`
- `internal/landingshed#Finalize.pullRequestLanding`
- `internal/landingshed#Finalize.dirtyParentReason`
- `internal/landingshed#resolveWithParentRule`
- `internal/landingshed#reconcileReport`
- `internal/landingshed#Finalize.syncParent`
- `internal/landingshed#Finalize.recheckLock`
- `internal/landingshed#Finalize.removeLeftovers`
- `internal/landingshed#Finalize.mergeAndGate`
- `internal/landingshed_test#queueHub`
- `internal/landingshed_test#newQueueHub`
- `internal/loomcli/doc.go`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#runReconciler`
- `internal/loomcli#writeLedgerLanding`
- `internal/loomcli#oldestHeldRecords`
- `internal/loomcli#landingParentTarget`
- `internal/loomcli#darnTreeScope`
- `internal/loomcli#parentFixture`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** A run with no outcome that holds the lock leads a batch: it builds the chain, gates the chain commit in its own task worktree and lands it with one guarded move and one ledger landing per run (D3, D4's use, D5's gate, D6, D10's second half).
A gate failure on a batch of more than one lands alone in this card; card 9 replaces that with the bisection.

New `Deps` fields, nil in a struct-literal test, which then takes no batch:
- `MemberDeps func(pairPath string) (Deps, error)` builds a member's own seams from its entry's pair path.
- `WriteLedgerLanding func(slug, tip, records string) error`, the ledger writer `writeLedgerLanding` over the parent target.
- `RecordsToHold func(records string) (string, error)`, the run's records commit to hold: the oldest one origin lacks, as `oldestHeldRecords` picks it, or records.
- `BuildChain func(base string, links []fabricengine.ChainLink) (fabricengine.ChainResult, error)` over the task pair.
- `FirstParent func(commit string) (string, error)` over the task pair.
- `Detach DetachDeps`: `WriteMarker func() error`, `RemoveMarker func() error`, `Checkout func(commit string) error`, `Restore func() error` over the task pair, through card 4's verbs.
`parentMerger` gains `MoveCodeTo(base, tip string) (bool, error)`, which `*fabricengine.Fabric` already has, and `recordingParentMerger` gains a recording `MoveCodeTo`.
`internal/loomcli`'s `landingDeps` fills them for the run at `l`; `TaskHead` there changes to resolve `refs/heads/<taskBranch>`, so every task-head read resolves the task branch, never HEAD (D5).
`memberLandingDeps(l, cfg, registry, runner)` resolves the pair path to its location, reads its task and parent branch and its recipe as `loomCLI.wireLanding` reads the run's own, and returns `landingDeps` for that location with the member's slug and the parent lock deps of the member's own status file.
It replaces the member's `Reconcile` with `judgeOnlyReconciler(location, slug)`: `runReconciler`'s reconcile with `fabriclayout.ReconcileScope{Run: slug, JudgeOnly: true}`, which fetches objects into no ref, moves no branch and tears nothing down, so the leader's read of a member acts on nothing (D6).
The env seam tables learn each new field, as card 7 does: `envkit.LandingDeps` fills them and the `Detach` funcs, `battencli`'s `intentionallyNil` gains each `Landing.<field>` and `Landing.Detach.*` path as "loom-only seam, batten lands no branch", and every reflective seam test card 7 names gains `//lym:seam internal/landingshed#DetachDeps`.

Taking the batch (D3), in `takeBatch`, after `actOnOutcome` answered none and `syncParent` passed:
a run with no queue entry, as one `joinQueue` skipped for a GitHub need or a failed read, leads no batch and lands alone (D10).
Otherwise the batch is the run's own entry first, then every `Queue.Entries` entry in join order that is `Live`, has no outcome, is not covered by a pause (the member's `PullRequest.PauseCovered`) and whose task branch tip (the member's `TaskHead`) equals its entry's `TaskHead`, up to `Config.BatchSize()` entries.
A member whose `MemberDeps` fails is skipped with a Warn.
A batch of one, a leader whose own outcome is `failed` or `conflict` (card 7 lands it alone before here), and a leader whose own `pullRequestLanding` sends it through a pull request, land alone through today's merge and gate (D8, D10).

Gating (D5), in `gateChainCommit`:
before the hold's first detach the leader runs `Detach.WriteMarker`; it checks the task worktree out detached at the chain commit, runs `removeLeftovers` measured from the commit it left and the clean check, and runs the batch gate.
The batch gate is `newBatchGate(deps)`: the Finalize gate key and command, no failure recorder, and an `onFlakyPass` that logs each failure the rerun did not repeat at Warn with the chain commit and appends nothing to `fz.flaky` (D4, D7).
`checkResult` is `check` returning the verify result too, and `check` calls it, so the landing gate keeps its behaviour.
On every way out, a failed gate and an error included, the leader runs `Detach.Restore`, `removeLeftovers` from the chain commit, and the clean check, and runs `Detach.RemoveMarker` after its last restore of the hold.
A restore that fails, as on untracked dirt the gate left, keeps the marker and ends the call Stuck naming the dirty paths, with the way forward "clean them in the task worktree, then `lym loom resume`", which card 10's shed restore then completes.
The task branch ref is never moved.
Bound: the gate config is the leader's own, so a run-level gate override on the leader applies to the whole batch, and a darn leader's tree scope diffs the merge base against the chain commit at HEAD.

Landing the prefix (D6), in `leadBatch` and `landPrefix`:
the leader builds the chain on the parent head with `BuildChain`; a `Conflicted` member gets outcome `conflict` and a `Held` one is left out with none.
After a passing gate and the restore, so no owner check reads a detached worktree, it runs its own owner checks and `recheckLock`, then `judgeMember` for each other member with that member's own `Deps`, as reads alone: the judge-only reconcile's ended, stopped and owned-elsewhere verdicts and an unreachable origin, `ParentOwner`, `LandBlockers`, `OwnedElsewhere` without a hand-over, `needsGitHubOnDisk`, and `resolveWithParentRule`'s decision, whose `Open` or `Halt` leaves the member out.
A check that would need an action (a hand-over, a follow-up removal, a task-branch fast-forward the judge-only reconcile reports, a pull request's opening, closing or record removal) leaves the member out with no outcome; its own Finalize meets the check under the lock.
Right before the move it re-reads each member's pause cover, `Live` and task branch tip.
A member left out rebuilds the chain without it and gates again; a parent whose head is no longer the chain's base rebuilds on the new head; one hold makes at most `Config.BatchSize()` rebuilds, then the leader lands alone through today's merge and gate.
Before the move it writes the move mark: the base, each prefix commit, and each member's slug and records commit, `RecordsToHold` of the member (the leader's own included) or the entry's `RecordsHead` when that read fails.
It then calls `MoveCodeTo(base, last prefix commit)`.
False removes the mark and rebuilds as for a moved head.
A `*fabricengine.MergeGuardError` removes the mark and ends the call Stuck with `dirtyParentReason`'s wording and way forward, as today's parent-side merge refusal does; any other error removes the mark and is returned.
Right after a move, still under the lock, it calls `WriteLedgerLanding` once per prefix entry with its own commit and the mark's records commit, sets each entry `landed`, and removes the mark.
It then does its own remaining landing records with its own landing commit: `recordLanding` gets `FirstParent(commit)` as the landed diff's base, `pushParent` gives the commit to `LandLocally`, and the lock is freed on the way out.

Tests.
`TestFinalizeQueue_LandsABatch`, tier1, over `queueHub` with each member's `MemberDeps` built from its real pair, members' entries joined with the test process as owner, a gate command that appends HEAD to a file, and a `WriteLedgerLanding` closure in the test that records each call with the parent lock record's holder it reads at that moment.
Rows:
- three queued runs land with one gate run, the parent's tip is the commit the gate recorded, and the ledger holds one landing per run with its own landing commit, each written while the lock record names the leader;
- the gate ran with the leader's worktree detached at the chain commit, which is left on its task branch with the ref unchanged after a pass and after a failed gate;
- a member that deletes a directory holding ignored build output in the leader's worktree gates and restores the leader clean;
- a gate command that commits on the parent pair on its first run, so the parent moves during the gate, leaves the parent unmoved by the leader and the chain is built again on the new head and gated once more;
- a gate command that moves the parent on every run, with `batch_max` 2, ends after two rebuilds with the leader landing alone through today's merge and gate;
- on a hub with `Config.Squash` false, each landing commit has two parents, the second its run's task head;
- after the batch each run's `RecordLanding` and `LandLocally` commit is its own landing commit, the leader's in its batch call and each member's in its own Finalize, whose landed size is measured from its chain commit's first parent, another member's landing commit;
- a member whose `RecordsToHold` fails gets its entry's `RecordsHead` in the mark and its ledger landing;
- a gate whose first run fails a test and whose rerun passes lands the batch, and the leader's landing commit message carries no flaky section;
- a batch takes at most `landing.batch_max` entries in join order, and skips a paused member, one whose task branch moved past its entry and one with an outcome;
- a leader whose entry carries `failed`, with two live entries waiting, lands alone and leaves both with no outcome;
- a run whose `landed` commit the parent no longer holds, after the test resets the parent, with another live entry waiting, leads a batch that lands both;
- a run with no queue entry, with a live entry waiting, lands alone and takes no member.
`TestFinalizeQueue_LeavesOutAMember` rows:
- a member whose reconcile reads it owned elsewhere, or ended on origin, after it joined is not moved and the others land;
- a member whose `LandBlockers` lists a blocker, or whose `ParentOwner` reads another machine, is not moved and the others land;
- a member whose task branch is behind origin, which its own reconcile would fast-forward, is left out with its branch ref unmoved;
- a member paused, or whose task branch moved, after the batch was taken is not moved;
- a member whose owner process dies after the batch was taken is not moved;
- a member whose `MemberDeps` fails is skipped and the others land;
- a member whose pull-request record appears after it joined is left out and then lands alone through its pull request, with the GitHub calls through `prkit`'s fake;
- a leader whose own landing goes through a pull request lands alone and takes no member;
- a parent pair dirtied during the gate is Stuck naming the dirty paths, with the parent unmoved and no mark left.
`TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed` gains a row: a member's records commit after the one its ledger landing holds is not sent, as `RunCommitsAhead` of the member reads it, until the parent's push removes the landing.
`TestDarnTreeScope_ClassifiesAtEachCall` gains a row: a darn worktree detached at a chain commit holding two members' work scopes its targets to both members' trees.
`TestLandingDeps_TaskHeadReadsTheBranchWhileDetached`, tier1 in `package loomcli`, pins that `landingDeps`' `TaskHead` reads the task branch's tip while the worktree is detached at another commit.

**ImpactSummary:** Adds six Deps seams and one parentMerger method, filled at their one builder, faked at their one fake and registered in the env seam tables; the leader path is new code inside Finalize's locked pass.
