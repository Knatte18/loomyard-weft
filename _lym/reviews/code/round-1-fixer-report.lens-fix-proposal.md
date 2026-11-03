```yaml
lens: fix-proposal
outcome: reported
```

**Task:** challenge the reviewer's proposals the fixer applied as proposed (F1, F3, F5, F6, F7 to F10), looking for cases where the proposal is wrong or breaks something.

**Findings**

1. **MEDIUM — `prepush.go` `guard()` parent-branch notice, combined with F6's extra guard in `Commit`.**
   - Summary: the parent-branch notice can repeat on every status transition and every `commit-records` call.
   - Evidence:
     - The notice fires on `parent.State == BranchFastForward || BranchDiverged`.
     - The fixer's own code treats these states as unmoved. Only `movedBranches` reads `Moved`, and only for the run's own branches.
     - `ownBranches` drops the parent branch because its `Slug` differs, so a run-scope reconcile may leave the parent at `fast_forward` or `diverged` until the operator updates it.
     - The notice has no memory, and F6 now runs `guard()` in `Commit` and again in `Spawn`, so each transition posts the line twice.
     - `commit-records` runs `Commit` then `Push`, and each of those runs `guard()`, so it posts twice more.
     - A run on a persistently stale parent therefore spams the orch for as long as it lasts.
   - Concrete case: a run on a stale parent, over 10 producer transitions, posts 20 identical "origin moved the parent branch main (fast_forward)" lines.
   - I did not find a test for the repeat; all new rows call `guard()` once.

2. **MEDIUM — `prepush.go` `stoppedBranches` and `stopReason`, "origin moved …" wording over the parent's local-condition states.**
   - Summary: a parent in `live`, `dirty` or `busy` state is named under "origin moved", though origin moved nothing.
   - Evidence:
     - F1's proposal says to name the parent whenever `State.Stops()`.
     - `Stops()` includes `BranchLive`, `BranchDirty`, `BranchBusy` and `BranchAnchorStopped`.
     - `fabricengine/originreconcile.go` documents `BranchLive` as "belongs to a pair that is live, which a reconcile never moves".
     - A run whose parent is a live prime pair, or a parent with a dirty path, has its own entry stopped by an unrelated own branch. The pause reason then reads `origin moved task (diverged), main (live): …`, which misleads the operator.
   - The fixer also added no test row for a parent in a local-condition state.

3. **LOW — `prepush.go` `guardedCommit` doc: "no push starts after it".**
   - Summary: the claim holds for the status seam only.
   - Evidence:
     - `commitRecordsVerb` calls `Push` after `Commit` regardless of the commit result.
     - After a stopped `Commit` the run is paused, so the second `guard()` skips on `pauseCovered`.
     - `PushPairAnchored` is then held only by the pause gate. `statusPushRequest` supplies `PauseSlug` at `pushanchored.go:163`, which I judged probably gated. I did not verify the gate.
     - If that gate does not hold, `commit-records` pushes earlier local commits after the stop and reports no `push_stopped`. That outcome is also untested.

4. **LOW — F5 proposal (`stopReason`, failed `landed()` returns `"", false`).**
   - Summary: the push goes for an entry origin ended, when `landed()` fails transiently.
   - Evidence:
     - `EntryEnded` means origin's board shows the run done or removed.
     - A transient `landed()` error now lets `Push` proceed for a run that origin removed, which can recreate its branch on origin.
     - The review chose this trade-off. It is not tested, apart from the new `EndedWithUnreadableLanding` row, which only asserts no stop.

No problems found with F3 (`skipsCheck` keeps the original read order), F7 rows, F8 or F9. F10 holds, since F2, F3 and F8 tests were added.

**Further tool calls (1 of 6)**
- `grep PauseSlug` and `sed` of `PushPairAnchored` in `internal/fabricengine/pushanchored.go`: my context lacked it, and I needed to know whether a synchronous push is held by the pause gate after a stopped `Commit`.
