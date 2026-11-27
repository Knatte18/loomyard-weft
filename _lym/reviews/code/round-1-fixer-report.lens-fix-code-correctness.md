```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed (fix-code-correctness).

1. **LOW**: `finalize.go` `Call`, new "Step 1b0" `notice := fz.configChangeNotice()`.
   - The notice is now built before `syncParent` fast-forwards the parent, so its "parent" side can be stale.
   - Take a local parent behind its upstream (F4's common case) whose upstream already holds the task's loom config value. `ReadConfigChanges` resolves `parentTip` from the local weft branch of the parent (`configchanges.go`: `revParseBranch(weftRepoRoot, parentWeft)`).
   - The notice then names that key with the old parent value and asks for it to be re-applied. This is contrary to `configChangeNotice`'s doc: "a key the parent already holds are not named".
   - It is also built before the owner checks' reconcile, which can fast-forward the task branch. Config commits that arrive that way are missing from the notice.
   - Before this fix, both reads happened after the reconcile and the sync. No test pins either order.

2. **LOW**: `deps.go` `ConfigChanges` doc ("so the diff's base is the task's fork point unless an earlier row merged the parent in").
   - The code does not hold this bound for a resumed Finalize. An earlier Finalize call that merged in its lock-free or locked pass and then halted (spent fixer, review cap) leaves the parent merged.
   - The next call's `merge-base(parentTip, tip)` is then that parent tip, not the fork point, with no earlier row involved.
   - The "Step 1b0" comment ("the base is the last parent tip merged in") is accurate.

I found no other defect.
- **Callers:** both `parentMergeTip` callers and the `gateUnlocked` and `landUnderLock` call sites match the new signatures. `ParentAlign` keeps `front=false`.
- **Owner checks:** `ownerChecks` in `gateUnlocked` ends the call correctly on a reconcile error, an unreachable origin, an ended or stopped run, a hand-over or a parent-owner read failure, all before `premerge.clear` or the merge.
- **`!held` guard:** when it skips the compare, the pass still merges and gates under the lock.
- **New tests, traced against the defect:**
  - The owner rows fail without the lock-free owner check, because a tip is recorded.
  - The push-skipped front row fails with `front=false` on the tip read.
  - The behind-upstream row fails if the compare runs before the sync.
  - The front test's `sleep` hook is synchronised through `pause`'s `slept` channel, so `fronts` has no race.
- **Not covered:** F6's `!held` and F7's read order have no test. Neither was reported as a coverage gap.
- **Integration tier:** `finalize_integration_test.go` is tier 1 and the gate run was tier 0, so it did not run. From reading its setup, it uses a lost-race bound of 0, and I saw nothing in it that the change would break.

Tool calls:
- One `head`, `sed` and `grep` over `finalize_integration_test.go` and the `loomcli` integration tests: the parent had not read how they wire `Reconcile`, `ConfigChanges` or the lost-race bound.
