# Card 9 — finalize-batch-bisection

**Create:**
- `plan:internal/landingshed#Finalize.bisectBatch` -> `func (fz *Finalize) bisectBatch(ctx context.Context, parent parentMerger, base string, chain []fabricengine.ChainCommit, members []batchMember, failed verifytree.Result, notice string) *ending`
- `plan:internal/landingshed#Finalize.failAsCulprit` -> `func (fz *Finalize) failAsCulprit(ctx context.Context, base string, result verifytree.Result) *ending`
- `plan:internal/landingshed_test#TestFinalizeQueue_BisectsAFailedBatch` -> `func TestFinalizeQueue_BisectsAFailedBatch(t *testing.T)`
- `plan:internal/landingshed_test#finalizingSeats` -> `type finalizingSeats struct`

**Edit:**
- `internal/landingshed#Finalize.leadBatch`

**Uses:**
- `internal/landingshed#Finalize.gateChainCommit`
- `internal/landingshed#Finalize.landPrefix`
- `internal/landingshed#Finalize.mergeParent`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#verifyGate.recordFailure`
- `internal/landingshed_test#queueHub`
- `internal/landingshed_test#newQueueHub`

**Intent:** A failed batch gate is bisected under the same lock (D7).

`leadBatch` hands a failure of a batch of n > 1 entries, after the batch gate's rerun of a test-run failure, to `bisectBatch`, in place of card 8's landing alone.
A result that is a runner crash or a memory-pressure stop halts the leader with the gate's `LandingStopPrefix` reason, as today, sets no outcome and bisects nothing.
`bisectBatch` builds no new chain: it searches the chain's prefixes, the empty one counting as passing and the whole chain as failing, gating the middle prefix's last commit through `gateChainCommit` at each step, until a passing prefix of k entries is followed by a failing one of k+1.
The marker stands across every step, since only the hold's last restore removes it.
Entry k+1 is the culprit and gets outcome `failed` with the chain commit it failed on; the entries after it stay with no outcome for the next leader.
A k of at least 1 lands that prefix through `landPrefix`, as D6 says, with its rebuilds; a parent moved meanwhile rebuilds the chain and gates again, as there.
When the culprit is the leader (k = 0), the leader lands nothing, sets its own outcome `failed`, and `failAsCulprit` runs: it merges the chain base into its task worktree under the lock through `mergeParent(ctx, base)`, which chain commit 1 shows clean, so the worktree holds the tree the prefix-1 gate failed on and the merge record names that base;
it writes its landing failure record from the prefix-1 gate's result through the landing gate's `recordFailure`, with the merge commit set, reads the attempt record and calls `runFixer` on it as `gateAndFix` does after a failing gate.
`runFixer` runs no gate before its first fixer turn, so no gate runs twice on one tree; it spends the attempt and reads the failure log from the record just written, unchanged.
The batch gate wrote no landing failure record or flaky record for the leader (card 8's batch gate has no recorder), so only a run's own gate on its own path, and a culprit leader from the prefix-1 log, write them.
Bound: a failure that comes only from two runs together blames the later one, whose own gate then runs on the parent holding the earlier; a failure from ungated upstream commits `syncParent` brought, a tier timeout or a build failure, which the rerun does not repeat, blames entry 1 or the prefix where it first shows, and the blamed run's own gate then runs on its own path, so nothing lands ungated.
A culprit's next Finalize sees `failed` and gates alone with its own fixer (card 7).

`TestFinalizeQueue_BisectsAFailedBatch`, tier1, over `queueHub` with a gate command that fails on a tree holding a marker file one member commits, and appends HEAD and whether the detach marker stands to a file at each run.
`finalizingSeats` is its `FixerSeats` fake: at its first fixer turn it builds and calls a second member's `Finalize`, records what that batch took, and answers the turn as spent.
Rows:
- a batch of four whose third run fails lands the first two with one ledger landing each, gives the third `failed` with its chain commit, leaves the fourth with no outcome, writes no landing failure record for the leader, and its gate file shows the detach marker standing at every step;
- a batch whose culprit is the leader lands nothing, gives the leader `failed`, leaves the other entries with no outcome, leaves the leader's worktree merged at the chain base with its failure record naming the prefix-1 log, and the gate file shows no run on that merged tree before the fixer's turn;
- while that leader's fixer runs with the lock freed, `finalizingSeats` runs a second member's Finalize, whose batch does not take the leader's entry;
- a gate command that exits as a memory-pressure stop on the whole batch halts the leader with `LandingStopPrefix`, gates no prefix and sets no outcome.

**ImpactSummary:** Replaces card 8's land-alone fallback in one method; `runFixer` and the gate are called as they stand, and nothing outside Finalize's locked pass changes.
