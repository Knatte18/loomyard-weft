```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the state each decision introduces or changes through the code paths that already write or read it.

**Findings**

1. **BLOCKING — D6 / D8: the parent stays ahead by unrecorded landings, and the owed-tip readers misread that.**
   - **What D6 does:** the parent is fast-forwarded to the whole chain, but only the leader records its landing at once. Each other run records its own later, when it takes the lock.
   - **How the ledger entry is written:** through `Deps.LandLocally(tip)`. Its wiring, `landLocally` → `recordHeldLanding` in `internal/loomcli/landingdeps.go`, commits the run's own status records under its pair push lock. Until each member does this, only the leader's tip is an owed tip.
   - **Readers that break:**
     - `pushledger.OwedTips` and `AheadByOnly`, read by `primeAheadByOwedTips` in `internal/fabricengine/pull.go`, need every commit past the tracking ref to be an owed tip.
     - `ParentAheadByOwed` in Finalize and in `parentalign.go` does the same.
     - `syncParent` reads such a parent as "ahead of upstream, not by owed tips" and returns Stuck.
     - The pusher pushes only up to the recorded tip. Its `payOwedSteps` and `AheadByOnly` checks judge the same commit list.
   - **Counterexamples:**
     - A run retaking a freed lock takes a Front ticket and goes ahead of the members still waiting. Its `syncParent` then finds L2 and L3 unrecorded and halts Stuck.
     - A member covered by a pause yields its ticket and never wakes. Its landing commit then stays unrecorded for as long as the pause lasts. Every later landing on that parent is Stuck, the prime's pull misjudges it, and that member's board entry is never marked done.
   - **Wrong name in the record:** D6 cites `Deps.RecordLanding` as the per-run record. That seam writes only the dobby's landed-commit scratch (`shedrun.RecordLandedCommit`), not the ledger entry the pusher reads.
   - **Fix:** settle who calls `LandLocally` for each member, and when. Every chain commit needs a ledger landing in the same step as the fast-forward, or before it, with each member's records commit coming from its entry. Otherwise the record must state a bound that the readers above tolerate.
2. **MEDIUM — D5: `Deps.TaskHead` reads the worktree HEAD.** Its wiring is `fabricengine.Open(l).HeadSHA()`.
   - Everything that reads the task head while the leader is detached gets the chain tip, not the task head. That covers `parentHoldsWork`, `landedDiffEmpty` and merge-record writes.
   - D5's restore-first-on-resume rule covers the crash path. It does not cover reads inside the gate-and-halve loop, such as the leader's own landing record or a halt path that reads `TaskHead`.
   - The record should state that every task-head read in the batch path resolves the task branch ref, not HEAD.
3. **MEDIUM — D6 / D8: `ParentHoldsTaskWork` is the already-landed test.** It is `Fabric.HoldsWork`: merge-tree(parent tip, task head) == parent tree.
   - For a member, that test passes for a chain landing, so the already-landed path is reachable.
   - The path skips the owner checks and, today, does not call `LandLocally`.
   - D8's "records its landing if the ledger lacks it" therefore means adding a ledger write to a path that has none. The record treats it as existing behaviour. Since this ties to finding 1, the record should name it as a change.
4. **LOW — D9: removing `landing.finalize_lost_races`.** Its only readers are `internal/landingshed/config.go` (the field and its below-0 validation), `keynames.go` (`finalizeLostRacesKey`), `doc.go` and `finalize.go`. No reader outside landingshed breaks, so D9 holds.
5. **LOW — D5: the gate caches.** The verify and green caches are keyed by tree, so a detached gate writes entries under the chain tree. When the leader goes back to its own branch, its tree differs and nothing stale is reused. This holds.

**Paths traced, per decision**

- **D1:** the lock wait (`parentGuard` → `parentlock.Acquire`) survives. The lost-race readers are confined to landingshed and survive.
- **D2:** parentlock's liveness model is reused and survives. The idle release (`parent_lock_idle_release_min`) bars a live verify and survives.
- **D3:** the pause yielding in the ticket queue survives at selection time. It breaks after selection; see finding 1.
- **D4:** `MergeTree` and `HoldsWork` survive.
- **D5:**
  - `TaskHead` via HEAD breaks; see finding 2.
  - The verify and green caches survive.
  - The task-pair pusher's view of a detached HEAD is unknown: I did not trace it.
- **D6:**
  - The `LandLocally` ledger write, `OwedTips`, `primeAheadByOwedTips`, `syncParent`, the pusher's `PushTip` and `payOwedSteps` all break; see finding 1.
  - `UnsettledLandings` (trailer-based, used by the pusher, pairteardown and beamcli) survives: the chain commits carry trailers.
- **D7:** no outside readers.
- **D8:** the `ParentHoldsTaskWork` already-landed path survives as a detection; its missing ledger write is finding 3.
- **D9:** only landingshed reads the key; survives.
- **D10:** the resolver is unchanged and survives.

**Tool calls**

1. I grepped for readers of `finalize_lost_races` outside `finalize.go`. My context lacked the key's readers.
2. I read `runpusher` `payOwedSteps` and `primeAheadByOwedTips`, and grepped `OwedTips`. My context lacked how the owed-tip readers judge a parent that is ahead.
3. I grepped `TaskHead` and detached-HEAD handling. My context lacked how the task head is read.
4. I read the `TaskHead` and `RecordLanding` wiring and `HoldsWork`. My context lacked what `RecordLanding` writes and how holds-work is tested.
5. I grepped the `LandLocally` wiring. My context lacked which seam writes the ledger landing.
6. I read the body of `landLocally` and `recordHeldLanding`. My context lacked how the ledger write holds the run's own records commit.
