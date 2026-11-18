```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, which traces each change in the diff to the board entry's goal points.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries.
- **G1:** reconcile from the run records themselves (a run reading running whose waiting process is gone), not from the status error text that `reconcileInterruptedStep` reads (`_lym/discussion/decision-record.md` Goal, "Idea" line).
- **G2:** a shuttle run left running by a step that wrote no interrupted record is reconciled on a re-step, so the re-step no longer ends on it (Goal, second paragraph).

**Per change → decision**

1. **`Runner.StopOrphanedStepRuns`** (`internal/shuttleengine/stepstop.go`, diff lines 473–482). It selects running runs whose step is under `stepsDir` and for which `!runIsLive(rs)` holds. **G1, G2.**
2. **`stopStepRunsWhere` / `stepRunCandidate` extraction, with `StopStepRuns` re-expressed through it** (`stepstop.go`, diff lines 467–570). **Forced by change 1**, because the stop and hold logic is shared.
   - Reed is now probed only after a candidate qualifies. This also changes `StopStepRuns` (diff line 468): it no longer reports a probe failure when it has nothing to act on.
   - That deferral is what keeps the per-loop-start reconcile from probing reed every time. Applying it through the shared helper makes it forced, not separate scope.
3. **`LoopSpec.ReconcileOrphanedSteps`, its call in `runLoop` before the first child, and the removal of `reconcileInterruptedStep` and `newestInterruptedTraceID`** (`internal/shedverbs/loop.go`, diff lines 218–301). **G1, G2.** The text check `strings.Contains(first.Error, " interrupted (")` is gone (diff line 254), and the hook runs whatever the status reads.
4. **loomcli wiring** (`arm.go` line 9, `cli.go` lines 21–24, `looparm.go` `orphanedStepsNote` / `stopOrphanedStepRuns`). **Forced by changes 1 and 3**: this connects the hook to the runner.
5. **Extraction of `stepStopNote` and `stepStopRunner`** (`looparm.go`, diff lines 76–180). **Forced by change 4**: the new hook and the existing ones now share rendering and runner construction.
6. **Comment in `loopwait.go` `loopExitStop`** (diff lines 391–393). **Forced by change 3**: the old comment said the record is what lets the re-step reconcile, which is now stale.
7. **Docs: `shedverbs/doc.go` (diff lines 193–200) and `shuttleengine/doc.go` (diff lines 445–449).** **Forced by changes 1 and 3.**
8. **Tests: `cli_test.go`, `loop_test.go`, `loopwait_test.go`, `stop_test.go`.** **Forced by changes 1–4**: new behaviour, plus removed symbols that the old tests called.
9. **Commits `3b6e4fd2` and `ffc0a7e7`.** They re-break comment lines in files the diff already touches (`git diff --stat` lists only the 12 files of changes 1–8). **Forced**: comment-lint upkeep of the changed comments.

**Per decision → changes**
- **G1:** changes 1 and 3, with 2 and 4–8 forced by them.
- **G2:** changes 1 and 3. Running on every loop start covers a step that wrote no interrupted record, and a step that ended in an error.

**Findings:** none. Every change traces to G1 or G2 or is forced by a traced change, and both goal points are built.

Tool calls: none. The inherited context already held the decision record, the diff, the commit log and the diff stat.
