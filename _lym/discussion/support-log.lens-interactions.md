```yaml
lens: interactions
outcome: reported
```

Interactions lens on the landing-queue decision record: whether decisions acting on the same state undo, outrun or contradict each other.

**Findings**

1. **BLOCKING, D7 with D3 (and the D7 acceptance criterion).** A non-leader can never become a "failing prefix of one", so outcome `failed` is never set.
   - D3 puts the leader's own entry first in every batch. D7 halves by gating the chain's first half, so every prefix starts with the leader, and a prefix of one is always the leader's own entry.
   - D7's branch "a failing prefix of one that is another run's entry" therefore cannot happen, and D8's `failed` branch is dead code.
   - The D7 criterion contradicts this. Take [A,B,C,D] where C is the fault. [A,B,C,D] fails, [A,B] passes and lands, and C and D stay queued with no outcome. C is not marked `failed`.
   - C is found only later, when it leads [C,D] and its own prefix of one fails into its own fixer.
   - Fix: either drop `failed` and say a culprit is found when it next leads, or define the halving to keep searching the rest of the batch after a passing prefix, e.g. gate [A,B,C] under the same lock. Then rewrite the D7 criterion to match.

2. **BLOCKING, D6 with today's ahead-by-owed rule (the Constraints cite it).** Members record their landings late, so the parent reads as diverged until they do.
   - D6 fast-forwards the parent to the chain tip. The leader records only its own landing, L1. Each member records L2, L3 only when it later takes the lock (D8).
   - `fabricengine.AheadByOnly` (`internal/fabricengine/landing.go`) passes only if every first-parent commit past upstream is an owed tip. `primeAheadByOwedTips` uses it, and the Finalize doc says the same: "lands on a parent ahead of its upstream by landing tips whose push is still owed, and on no other ahead parent".
   - Until every member has recorded, the parent is ahead by commits that are not owed tips. The parent pusher's reconcile, prime pulls and catch-up, and the next Finalize's `syncParent` will all read it as ahead or diverged and refuse or go Stuck.
   - A member that died before waking never records, so this lasts.
   - Fix: the leader writes every member's ledger landing before or with the fast-forward. That means each entry must carry what `RecordLanding` needs, the records commit included, or there must be a ledger "landing without records commit" that the member completes later.

3. **MEDIUM, D2 with D6/D8 (restart).** Removing dead owners' entries destroys the outcome that recovery relies on.
   - D2 says the next leader removes a dead owner's entry. D6's crash recovery ("a run that wakes and finds its planned landing commit records it if the ledger lacks it") reads the planned commit from that same entry's outcome.
   - Case: a member crashes while waiting after the leader wrote `landed` and fast-forwarded. The next leader removes the dead entry. The resumed member then has no outcome, and D1 rewrites its entry.
   - Today's already-landed path (`ParentHoldsTaskWork`) still ends the run, but its missing ledger record is never written. That compounds finding 2.
   - Fix: keep entries whose outcome is `landed` until their run records, or find the landing commit from its `Lym-Landing` trailer.

4. **MEDIUM, D8 with D3.** A run returning from `failed` or `conflict` is told to land alone, but D3 would make it a batch leader.
   - D8 says such a run "clears the outcome… lands alone". D3 makes any lock holder whose entry is unresolved a leader that takes a batch.
   - Once the outcome is cleared, D3 applies, so whether the run batches others is contradictory.
   - Fix: say which one wins, e.g. the run lands alone and leaves its entry out of the queue until it has landed.

5. **LOW, D6 with D7.** D6 says the leader writes `landed` into "each entry". When a prefix passes, D7 leaves the entries after it with no outcome. D6 should say "each entry of the landing prefix".

6. **LOW, D6/D8.** If a leader dies after writing `landed` but before the fast-forward, a member that takes the lock next sees `landed` while its commit is not on the parent. D8 lists only "landed with its commit on the parent". D6 covers the case only for a leader, so D8 should send this member to its D3 path explicitly.

7. **LOW, D7 with D3 (the leader as culprit).** When the leader's prefix of one fails and it goes to its fixer, D7 does not say what happens to the rest of its batch. Presumably the entries stay with no outcome for the next leader, which the fixer allows by freeing the lock. State it.

**Pairs checked**

- D1/D3, a lone run leads a batch of one under the lock: held.
- D1/D10, the pull-request decision moves before the entry, off the lock: held.
- D1/D4, a member's head merged an older parent tip, and merge-tree uses the right base: held.
- D2/D6, dead-entry removal against `landed` recovery: broken (finding 3).
- D3/D7, leader first against halved prefixes: broken (finding 1).
- D3/D8, a cleared outcome against leading: broken (finding 4).
- D3/D9, `batch_max` 1 means a batch of one: held.
- D3/D10, a pull-request run writes no entry and lands alone holding the lock: held.
- D4/D7, a conflicted entry is left out and the chain rebuilt: held.
- D5/D7, several detached checkouts during bisection, restored on exit: held.
- D5/D6, resume restores the worktree, then the already-landed path runs: held.
- D6/D8, landing recorded late against ahead-by-owed: broken (finding 2).
- D6/D7, which entries get `landed`: broken (finding 5).
- D6/D8, `landed` while the parent lacks the commit: broken (finding 6).
- D7/D8, a culprit's path: dead (finding 1).
- D9/the idle-release constraint, the leader's live verify keeps the lock: held.
- The D7 criterion against D3/D7: broken (finding 1).
- The D6/D8 criterion ("each run records its own landing") against the ahead-by-owed constraint: broken (finding 2).
- The other criteria against their decisions: held.

**Tool calls**

- grep of `primeAheadByOwedTips`: I needed whether "ahead by owed" judges each commit or only the tip.
- grep of `AheadByOnly`: I needed the rule's exact per-commit check, confirmed every first-parent commit.
