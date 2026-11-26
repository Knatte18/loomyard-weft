```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code and tests in the fix's change set (fix-code-correctness lens).

1. **MEDIUM**: the new `TestRun_DoneOutcome` row "Master's write to the friction note its prompt names leaves the run done" (`internal/websterengine/runlevel_test.go`) passes with the defect in place.
   - Its note lives in `t.TempDir()`, outside the worktree. A parent write there is only a policy finding (`recordbatch_test.go`, "a parent write outside the worktree is a policy warning"), which leaves the run `done` and adds a warning.
   - So if run exit passed `""` to `CheckParent` instead of `st.MasterNotePath`, or `st.MasterNotePath = masterNotePath` were dropped, `result.Outcome` would still be `done`. The row checks nothing else.
   - F5's run-exit reader therefore stays unpinned. The row needs to assert that no `parent-write` warning is in `result.Warnings`, or use a note under the worktree's `_lym`.
   - The record-batch row does check `Warnings`, and the `TestRun_MasterSpawn` row pins the save itself.

2. **MEDIUM**: taking over the earlier pre-fix head (`earlierPreFixHead`, and the `readEarlier` block in `newVerifyGate`) also applies the commit check to everything committed between the two runs.
   - Example: run 1's gate fails and the run ends stuck. The parent then does a conflict-resolved parent merge-in on the task branch (not a clean merge) and re-runs.
   - `earlierPreFixHead` accepts the old head, since it is on HEAD and the last batch head is below it. `fixRejection(preFix)` then fails the gate `Terminal` on that merge. Before the fix this run's closure started with an empty `preFix` and verified normally.
   - `PreFixHead` stays persisted, so every re-run fails the same way. The only way forward the stuck reason names is `reset --to pre-fix`, which now drops the parent's between-run commits.
   - The reviewer's F2 proposal implies the wider check, but not this dead end. The `NewVerifyGate` doc comment says nothing about it either.

3. **LOW**: the `IgnoredPath` call kept in `recordsSideStatus` (`findingtext.go`) no longer changes any result, and the new `Ignored` row passes without it.
   - An absent path is never reported ignored.
   - A file tracked at `base` is not reported ignored while it stays tracked.
   - A present ignored file that `base` does not hold already compares `now != ""` against `then == ""`, so it counts as changed.
   - The F14 row therefore does not pin the branch it was added for.

I found nothing else wrong in these, traced by hand:
- the `walkParentMerges` / `reconcileHead` refactor;
- the `headMismatchError` handling in `reconcileAcceptHead`;
- `recordGatePass`'s range joining;
- the changed `recordPass` and `recordsSideStatus` signatures and their callers;
- the `reset.go` clear.

Further tool calls: none.
