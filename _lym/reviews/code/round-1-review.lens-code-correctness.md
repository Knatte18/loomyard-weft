```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** I traced each function and branch the diff adds or changes with a concrete input.

**What I ran, and whether it holds**

1. `Call` loop with `FinalizeLostRaces` = 2 and the parent unmoved. The lock-free pass merges and gates tip p1. The locked pass finds head == `gatedTip`, the locked merge is up to date, and verifytree skips the tree it already verified. It lands. Correct.
2. `Call` with bound 2 and the parent moving once: `lostRaces` is 1, the pass runs again lock-free on p2, and it lands. With bound 1, `lostRaces` reaches 1 == bound, `gatedTip` is reset to "", and it merges and gates under the lock. Correct.
3. `Call` with bound 0: `0 &lt; 0` is false, so the lock-free pass never runs and the behaviour is as before. A hub config without the key reads the template default of 2. The other keys' "below 1" refusals only make sense if template defaults fill missing keys (`config.go:239-251`, `template.yaml`). Correct.
4. `Call` where the fallback locked pass frees the lock for a fixer. `lockFreed` makes it return nil, the loop runs again, and `lostRaces` stays at the bound, so it stays locked. Fixer attempts cap the loop. Correct.
5. `gateUnlocked` with the held read failing: owner halt. With held true: nil, no tip kept. Correct. With `parentMergeTip` failing: `fail`. With a lock reason: `halt`. Correct. With a merge conflict, Stuck, or a gate failure: the ending propagates. Correct.
6. `gateUnlocked` → `runFixer` → `freeLock` with no lock held: a no-op, and `lockFreed` stays false. The fixer's change is let through and `gatedTip` = the pre-fixer tip. The locked compare still catches a parent that moved while the fixer ran. Correct.
7. `parentMergeTip`, upstream with relation Equal or Behind: it takes `Upstream`. Under the lock, `syncParent` fetches again and fast-forwards when Behind, so head is the latest upstream. An unchanged upstream matches; a moved one sets `parentMoved`. Correct.
8. `parentMergeTip`, relation Ahead or Diverged: it merges and gates against the older upstream. `syncParent` then stops Stuck under the lock. That Stuck is reached only after a full lock-free merge, gate and possibly fixer (finding 1).
9. `parentMergeTip`, no upstream or push skipped: it takes the lock to read `tips.Local` and releases it. Under the lock, head (`HeadSHA`) == local tip. Correct. It queues with `front=false` on every pass (finding 3).
10. `parentMergeTip` gets `guard` by value: `parentGuard` holds no mutable state. The ParentAlign refactor is behaviour-neutral apart from the producer name in error text. Correct.
11. `landUnderLock` step 1b4b, when `HeadSHA` fails: `fail`, and the deferred release runs. Correct. On head != `gatedTip`: `parentMoved` is set, it returns nil, and the lock is released by the defer. Correct.
12. `landUnderLock` where held is false in `gateUnlocked` but true after the owner checks' reconcile. `syncParent` is skipped and head is compared with `gatedTip`. A mismatch counts a "lost race" that is no race, then the next pass takes the held path (finding 4).
13. `landUnderLock` `lockHeld`/`defer`: `lockHeld` = `Store != nil` only after `ensure` succeeds. When `freeLock` has already released, the defer calls `release` again, which is harmless because release leaves another owner's record alone. Correct.
14. `freeLock`: called twice in one pass (fixer, then fix review), the second call does nothing because `lockHeld` is false. This matches the old `lockFreed` short-circuit. With no store, nothing is set, as before. Correct.
15. `mergeAndGate` → `keepParentHead` now runs before `premerge.clear` and the clean check, which used to precede it. The head is read under the lock either way, so the landed-diff base (`parentHeadMerged`) is the parent head. Correct.
16. `mergeParent(tip)` and `judgeParentMerge(tip)`: with a tip set, `MergedTip` = the upstream SHA and `ReadParentTips` is not read. With "", it reads the local tip as before. Correct.
17. Locked up-to-date merge after a lock-free merge: `WriteMergeRecord` replaces the lock-free record with `UpToDate:true`, `MergeCommit:""` and an empty `Overlap`, and `recorder.mergeCommit` becomes "". Old passes after a fixer already did the same, and the fixer reads only `MergedTip`. The skipped gate never writes a failure record. Not a defect.
18. `rebaseAfterParentMerge` across passes: the lock-free merge record has `UpToDate:false` and `TaskHead`=Judged, so it rebases. The locked record has `UpToDate:true`, so it does not rebase twice. Correct.
19. `configChangeNotice` after the lock-free merge: `ReadConfigChanges` takes merge-base(parent weft, task weft). After the merge, that is the merged parent tip, not the task's fork point. The keys named come out the same; the reported "base" value is now the parent's value (finding 5).
20. `LoadConfig` with `finalize_lost_races: -1`: refused, naming the key through `keyName`. With 0: accepted. Correct.

**Findings**

1. **BLOCKING, design.** `internal/landingshed/finalize.go:309-317`, `gateUnlocked`. The lock-free merge, conflict session, landing gate, fixer and fix review now all run before the owner checks, the pull-request decision, the beam land check and the parent-pair checks.
   - A run whose origin ended or stopped, or that another machine owns, now commits merges and fixer changes on the task branch and spawns agent sessions before Finalize judges it.
   - The same holds for a parent another machine owns, a parent ahead of or diverged from its upstream, a dirty or mid-merge parent pair, and a pull-request "yes" without a record. Before this change each of these halted before touching the task worktree.
   - The pusher skips its owner check at Finalize because Finalize "judges origin and its owners itself", so those commits reach origin unchecked. This breaks PATTERN-owner-machine's pushing rule.
   - Evidence: `finalize.go:309-317` against `landUnderLock` at `finalize.go:399-438`; `internal/loomcli/prepush.go:89,108`; `pattern/PATTERN-owner-machine.md`, "Owner machine".
2. **MEDIUM, design.** `internal/landingshed/finalize.go:342`, with `parentalign.go:523-528`. With the push skipped or no upstream, every lock-free pass takes the parent lock for the tip read with `front=false`.
   - A run returning after a lost race or a fixer queues at the back behind every waiting landing, then queues again at the front.
   - This contradicts "Each later pass queues at the front" (`finalize.go:304`). On a busy parent it can add a whole queue wait per pass.
3. **LOW, design.** `internal/landingshed/finalize.go:335-341` and `442-451`. Held is read in `gateUnlocked` before the owner checks' reconcile, which can fast-forward the task branch.
   - A run that becomes "held" in the locked pass still runs the `gatedTip` compare. A mismatch frees the lock and counts a lost race that was no race, which spends the bound and wastes a pass.
   - Evidence: `finalize.go:406-423` (the reconcile can make held true), `finalize.go:442`.
4. **LOW, design.** `internal/landingshed/finalize.go:454-457`. `configChangeNotice` now always runs after the lock-free merge.
   - `ReadConfigChanges`' merge-base is therefore the merged parent tip, so the notice's "base" value is the parent's value. Step 1c's comment "the diff is taken against the parent as the task forked from it" no longer holds.
   - In recipes with no Parent-Align row (darn, sampler), the first pass used to see the true fork point.
   - Evidence: `internal/fabricengine/configchanges.go:76-80`; `finalize.go:454-456`.

**Tool calls**

- Read `internal/fabricengine/merge.go` `MergeIn`/`MergeInAt`: the manifest lacked how a tip-pinned merge treats the code and records sides.
- Grepped for the freshness rule and read `LoadConfig`/`configtemplate.go`: the manifest lacked the by-name merge's source pick and how a missing `finalize_lost_races` key loads.
