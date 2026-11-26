```yaml
lens: coverage
outcome: reported
```

I reviewed the plan through the coverage lens: does every decision, acceptance criterion and new branch have a test row that would fail if it broke?

**Findings**

1. **MEDIUM — D2, a failed read keeps the last level.** Card 3 step 1 says a failed `Level` read keeps `watchLevels[slug]`. No `TestRunWatch` row covers a read that succeeds and then fails. The only level row covers a level that has never read. Evidence: `03-run-watch-engine.md`, step 1 and the row list ("a level that has never read posts nothing…"). Fix: add a row where the level reads `rows`, then the read fails on a tick that has a row move, and that move still posts a line under the kept `rows`.

2. **MEDIUM — D2 acceptance, the refusal names the key's three values.** The acceptance criterion wants the per-task check's refusal to name the key and its values. Card 1's new rows check only `run_watch`, `all` and `lym config beam --set run_watch=`, so a refusal listing the wrong values would still pass. Evidence: `01-run-watch-setting.md`, Tests bullets 2 and 4. Fix: add `[stops rows off]` to both rows' expected texts.

3. **MEDIUM — D2, `CheckLocalValues` has no reaching row.** Card 1 adds `RunWatchKey` to `CheckLocalValues`, the `CheckValues` hook in `internal/configreg/configreg.go:151` that judges a task's `run_settings`. No test calls it anywhere in the repo: grep finds no `_test.go` caller. A `run_watch` left out of its key list would ship unnoticed. Evidence: `internal/dobbyengine/config.go:141`, `internal/configreg/configreg.go:151`. Fix: add a table test of `CheckLocalValues` with a bad `run_watch: all` row and a good `run_watch: rows` row on card 1.

4. **MEDIUM — D1, the child-read branches.** Card 3 step 3 reads the child only when `StatusModified` or `ChildStatusModified` moved. It also logs a failed child read under `<slug>|watch-child` and keeps the last child part. No row covers either branch: an unchanged mod time with a changed fake child (the child must not be re-read), or a failing `ReadChild`. Evidence: `03-run-watch-engine.md`, step 3 and the Tests list.

5. **MEDIUM — D3, paths render relative to the task worktree.** D3 says paths under the task worktree render relative to it. No row has a child or batten reason containing a path under `PairDir`, so a missing `pairRelative` call passes. The "cut to one line" rule has no row either (a multi-line reason). Evidence: `03-run-watch-engine.md`, step 5 and the row list.

6. **MEDIUM — D4, entry upkeep.** D4 drops an entry only for a run whose batten status no longer exists and that owes no line. A run left out of one tick's list because its status is unreadable keeps its entry. Card 3's upkeep (unlisted, `StatusExists` false, nothing owed, then drop; a failed or nil `StatusExists` keeps it) has no row. Neither the drop, nor the keep while `StatusExists` is true, nor the keep while a line is owed is asserted. Note that `passFake.statuses` defaults to false, so the harness drops by default. Evidence: `03-run-watch-engine.md`, "Entry upkeep"; `internal/dobbyengine/passes_test.go:202,223`.

7. **MEDIUM — D5, no run tag on a watch line.** The acceptance criterion says a posted watch line carries no topic and no run tag. Card 4's test checks the sender, the autonomy stamp and no supersede. The no-supersede check can catch a stray topic, but nothing would catch a run tag, which would let teardown drop the landed line D5 protects. Evidence: `04-run-watch-wiring.md`, Tests. Fix: assert the mail's run tag and topic are empty, or show a run's teardown drop leaving the watch line standing.

8. **MEDIUM — D6, the tier timeout mapping in the wiring.** Card 7's `dobbyGateHolders` maps `cfg.Tiers[holder.Tier].TimeoutMin` to `TierTimeout`, gives zero for an empty or undeclared tier, and wraps a failed gate config load or `Holders` read. No test reaches it, so a wrong unit or key goes unseen. The engine rows feed `TierTimeout` in directly. The criterion's "a failed gate config load … posts nothing and warns" reaches only the engine-side `Holders` error. Evidence: `07-gate-hold-wiring.md`, Intent paragraph 1 and Tests.

9. **MEDIUM — D6, unreached branches in card 6.**
   - A failed `Tree` or `MarkedPIDs` still posts, naming the tree as unreadable; no row.
   - A failed `TaskWorktree` skips the run with a Warn; no row.
   - A failed post is retried at the next probe tick and the key is recorded only after it succeeds; no row.

   Evidence: `06-gate-hold-engine.md`, Intent paragraphs on the step and the post, and the row list.

10. **LOW — D6, the check runs only on probe ticks.** No row asserts that a non-probe tick calls no `Holders` and posts nothing, although D6 and the PATTERN-daemon-wakeups constraint both say the check runs only then. Evidence: `06-gate-hold-engine.md`, "Tests: … over `newPassHarness` probe ticks".

11. **LOW — D1, the watch runs only on scheduling ticks.** No row shows a standing anchor move holding the watch lines and posting them once it ends. Evidence: `03-run-watch-engine.md`, "The step".

12. **LOW — D3, the child's shed done as a stop.** No row has a child going from running to done and posting one line under `stops`. The planned rows cover a batten block over a done child and the row moves after the done. A cancelling derived state has no row either. Evidence: `03-run-watch-engine.md`, the row list.

13. **LOW — D4/D5, a held clear that then clears.** The `TestClearDoneRuns` row asserts a run that owes a line is not cleared. No row asserts the run is cleared once the line posts or is dropped. Evidence: `03-run-watch-engine.md`, last Tests sentence.

Tool calls beyond my context:
- Grepped `CheckLocalValues(` across the repo: I needed to know whether any test covers the per-task check card 1 extends.
- Grepped `CheckLocalValues` without the paren: the first grep found no caller, so I located its registration in `configreg`.
