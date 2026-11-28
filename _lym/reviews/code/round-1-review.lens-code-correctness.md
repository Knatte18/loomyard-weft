```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `parentMergeTip` and `gateUnlocked` changes.

**Cases run**

1. `parentMergeTip`, `ParentAhead`, `takeOwedLocal=false` (Parent-Align): returns `tips.Upstream` and takes no lock. Correct.
2. `parentMergeTip`, `ParentAhead`, `takeOwedLocal=true`, nil `ParentAheadByOwed`: returns `tips.Upstream`, which matches the "Nil reads as no owed tip" contract at `deps.go:185`. Correct.
3. `parentMergeTip`, `ParentAhead`, flag set, seam returns an error: the error is wrapped with the producer and branch, and `gateUnlocked` sends it to `fail` before any merge. The wording matches `syncParent` at `finalize.go:725`. The seam reads local git only, so it carries no transport class. Correct.
4. `parentMergeTip`, `ParentAhead`, flag set, owed=false: returns `tips.Upstream`, and the locked pass's `syncParent` then refuses at `finalize.go:731`. Correct.
5. `parentMergeTip`, `ParentAhead`, flag set, owed=true: falls through to `guard.ensure`, reads `ReadParentTips(false)` under the lock and returns `tips.Local`. This is correct while the parent stays unchanged. Four interleavings between the lock-free owed read and the locked local read:
   - The owed push completes: local equals upstream, the tip is right, and `syncParent` sees `ParentEqual`. Correct.
   - Another landing lands during the lock wait: it is merged and recorded under that lock, at `finalize.go:555-559`, so the local read sees it whole, and the new tip is owed too. Correct.
   - A landing is half made (merged, not yet recorded): it cannot be seen, because that landing holds the lock. Correct.
   - A plain local commit (not a landing) lands on the parent between the owed read and the lock: the local read returns it, and the task worktree merges it. **Defect**, finding 1.
6. `parentMergeTip`, `ParentAhead`, flag set, owed=true, `guard.ensure` returns a pause reason or an error: the reason or error is returned before any merge. Correct.
7. `parentMergeTip`, `ParentAhead`, flag set, owed=true, the locked `ReadParentTips(false)` fails: the error is wrapped, nothing is merged and the lock is released by the defer. Correct.
8. `gateUnlocked` with owed=true and a landing that needs GitHub (a pull request is recorded, or the mode is trail): `Finalize.Call` runs `gateUnlocked` at `finalize.go:341` before `landUnderLock`. The `parentPushBlocks` check at `finalize.go:423-427` only runs inside `landUnderLock`, so the owed tips are merged into the task branch first, and the landing then ends with a transient error. The old code merged the upstream tip here. **Defect**, finding 2.
9. `gateUnlocked`, owed=true, gate fails and the fixer runs: the merge record's `MergedTip` is the local tip and the overlap is measured against it, at `finalize.go:1401-1422`. The values are consistent, and the next pass re-reads the tip. Correct.
10. Locked pass after an owed-local gate: `syncParent` re-reads `ParentAheadByOwed` and passes, `HeadSHA` equals `gatedTip`, and `mergeAndGate` merges `""` (the local tip, already up to date) and skips the verify. Correct.
11. `ParentAlign.Call` passes `false`, so its behaviour is unchanged. Correct.

**Findings**

1. **MEDIUM**, class design, `internal/landingshed/parentalign.go:251-274`.
   - **Summary:** the owed-only check runs with no lock, but the local tip it licenses is read later under the lock, and nothing re-checks the owed state at that read.
   - **Consequence:** a non-landing commit made on the parent between the two reads, such as an operator commit in the parent pair, is merged into the task branch. The locked pass's `syncParent` (`finalize.go:722-731`) then refuses the parent as ahead. By then the task branch already holds that unpushed commit, so it can reach origin through the task branch, or land again after the operator drops or rewrites it.
   - **Fix:** re-check `ParentAheadByOwed` under the lock, against the local tip it reads, and take `tips.Upstream` when the parent is no longer owed-only.
   - **Evidence:** `parentalign.go:251-257` and `parentalign.go:265-274`; `fabricengine/landing.go:207-218`.
2. **LOW**, class design, `internal/landingshed/finalize.go:380`.
   - **Summary:** for a landing that merges on GitHub, `gateUnlocked` now merges the parent's owed, unpushed landing tips into the task branch before `parentPushBlocks` ends the call.
   - **Old behaviour:** the GitHub path never merged owed tips while a push was owed. The lock-free pass took the upstream tip, and the locked pass blocked at `finalize.go:423-427` or `finalize.go:453-457` before `mergeAndGate`.
   - **Consequence:** if the owed push is later refused and the parent is reconciled by a rewrite, the resumed run pushes a task branch holding the stale landing commits through `pushTask` (`finalize_pr.go:474`), and GitHub merges them in.
   - **Cost:** the lock-free gate is also spent on a tree that cannot land.
   - **Fix:** pass `takeOwedLocal` only when the landing does not need GitHub (`!fz.needsGitHubOnDisk()`), or run the `parentPushBlocks` check before the lock-free merge.
   - **Evidence:** `finalize.go:340-345`, `finalize.go:380`, `finalize.go:422-427`; `finalize_pr.go:474`.

I made no tool calls.
