```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran concrete cases through the changed functions in the diff. One finding came out.

**Case list**
- `gitrepo.DropRemoteTracking` (`push.go`), tracking ref and upstream config both absent: correct. `update-ref -d` on an absent ref exits 0, and `config --unset` on an absent key exits 5, which matches `configKeyNotSetExit = 5` in `branchmark.go:21`.
- `gitrepo.PushBranch` and `pushPlainWithRetry`, marked branch, and a named branch with a retryable refusal: correct.
- `DeleteRemoteBranchLeased`, lease refused: returns `ErrLeaseRejected`. Correct.
- `PushWithRetryReasons` with "missing necessary objects": retried only for the archive-tag push, because that is the only caller passing extra reasons. Correct.
- `archiveWeftTip`:
  - origin absent: correct. The branch is pushed and the lease tip is the local tip.
  - origin behind the local tip: correct.
  - origin ahead of the local tip: correct. A second tag goes at origin's tip.
  - origin diverged: correct. The branch push is rejected and only logged, and both tips are tagged.
  - branch only on origin: correct.
  - sweep (`pushBranch` false): correct.
  - failed tip read or fetch: correct. The error is returned and the removal keeps everything in place.
- `Remove` with the origin-only path, no origin, and a local pair: correct. The empty lease tip means no delete.
- `deleteWeftBranchOnRemote`, lost lease: reported as kept in `RemoteKeptReason`, not `RemoteError`. Correct.
- `lockPairTeardown` callers:
  - Callers: `Add`, `DiscardTaskBranch` and the sweeps take the lock.
  - Nesting: nothing nests it. `cancel.go:663` calls the removal and `cancel.go:681` the discard one after the other, each with its own lock.
  - Warning gating: `LeaveClaim` is true at `cancel.go:663` and `beamfork.go:133`, so neither gets a spurious warning.
- `PushPairAnchored` shared lock: correct. A held exclusive lock gives `ErrPairPushRunning`, and the `fabric push` and `commit-records` callers turn that into an ok envelope with `push_skipped`.
- `cleanupOrphan` and `deleteLeftoverUnderLock`:
  - Lock held: skipped with a reason.
  - Lock error: reported in `Error`.
  - Pair turned live under the lock: no entry, or a kept reason.
  - Correct in each case.
- `Add` adopt-only:
  - Weft not live: `ErrNotLive` before any mutation. Correct.
  - Task branch gone: `ErrNotLive`. Correct.
  - Records worktree reused: only the task worktree is made, and rollback leaves the reused one. Correct.
  - Behind weft branch: fetched by SHA and merged `--ff-only` inside the reused worktree. Correct.
  - Reused worktree not counted as "another" by `refuseAdoptLocalCheckedOut`: correct.
- `dropStaleOriginTracking`: runs only when a probe ran and read the branch absent. Correct.
- `readoptPermanent`, `failReadopt`, `readoptWayForward`:
  - Closed-set split: correct.
  - Busy run lock, status read failure: transient, correct.
  - Non-`running` status: keeps state, correct.
  - Land restore: tag chosen by latest committer time among `done` children, correct.
- `refuseContinuePass`: refuses a gone worktree. Correct.
- `BattenFollowUp`, `cancelStoppedRefusal`, `preflightRunOwner`, `withFollowUp`, `reentryAfter`: correct. Finalize's held-parent path passes a nil follow-up, which is the D3 exception.
- `originTargets`:
  - Worktree not registered, option set: correct. The slug joins `pairs`, no branch is judged, and it reads as absent.
  - Worktree not registered, option unset: today's error. Correct.
  - Worktree registered but prunable: wrong. See F1.

**Findings**

F1 — MEDIUM, design, `internal/fabricengine/originreconcile.go:1061-1071` with `:663-672`

Under `PairMayBeAbsent`, the first resume reconcile of a run whose task worktree is registered but prunable skips the absent-pair handling. `registered` is true because it is computed from `warpEntries` without filtering `entry.Prunable`, so the code takes the `default:` branch. Line 663 then skips that prunable entry, so the slug is never appended to `pairs`. The run's entry is therefore never classified, and `PairAbsent` stays false.

`resumeRefusal` finds no entry, so owned-elsewhere, ended and stopped are never judged. `readoptGonePair` then proceeds to prune and `Add`. The parent branch is also judged, although D1 says it is not judged while the pair is absent.

This is the stale-registration case D1 itself lists (a deleted worktree not yet pruned), and it breaks `PATTERN-owner-machine`'s reconcile-first rule.

Fix: compute `registered` over non-prunable entries only, by adding `!entry.Prunable &&` inside the `ContainsFunc` predicate. A prunable pair then takes the `PairMayBeAbsent` branch. Add a test that deletes the task worktree directory without pruning and asserts `PairAbsent` and an entry verdict.

Tool calls:
1. Bash grep for `configKeyNotSetExit`, callers of `DiscardTaskBranch` and `removeWeftWorktree`, and `LeaveClaim`: to check the exit code and the lock nesting; the manifest lacked these callers.
2. Bash grep for `Prunable` in the worktree-list and reconcile files, plus `ExitCode` uses in gitrepo: to check how stale registrations are handled.
3. Read of `originreconcile.go:655-700`: to confirm that a prunable entry is skipped before the slug joins `pairs`.
