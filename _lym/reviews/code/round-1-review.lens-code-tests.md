```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour: the test that pins it**

1. **Board read bounded at `landingWalkStart`, older commits unread.** Pinned by `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn` (`landings_test.go:831`, reads `{edge, bd1, bd2}`).
2. **Board in effect at the floor is read as the oldest commit's first parent.** Pinned by the same test: the landing kind `feature` comes from `edge`, and dropping that read gives kind `other` plus a finding.
3. **Board in effect at the floor when no board commit lies inside the window (`before = tip`, `boardhistory.go:46-53`).** None.
4. **Floor lowered to the oldest time-join window's opening (`landings.go:149-154`).** None.
5. **`doneAfter` finds the first done commit after the opening.** Pinned by the existing `TestCompute_Landings` time-join rows (`landings_test.go:162-181`, `289-341`), with the same semantics as the old walk.
6. **`labelsAt` gives the newest labels at or before the landing.** Pinned by `TestCompute_Landings` kind rows (`landings_test.go:123`, `373-377`) and by test 1 (`edge` beats `old1`/`old2`).
7. **`add` reads only the first entry per slug, and `done[slug]` dedups (mutation survivor `boardhistory.go:101`).** No test, and none needed: dropping the dedup only appends a duplicate one-commit span, which `doneAfter` reads the same way. No finding.
8. **No parsed board is held by the joiner.** Pinned by `TestLandingJoiner_HoldsNoParsedBoard`. It would fail if `boardHistory` or `landingJoiner` held a `boardengine.Task`.
9. **Records walk opens `firstBattenLookback` before the window.** Pinned by `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan` row 1: `b0`/`b1`/`b2` are read 0 times.
10. **Seed decided at the oldest walked commit by reading its first parent (`isFreshSeed`, the k==0 branch).** Pinned by row 1: `gap` is read once.
11. **`battenStatusReader` reads each commit at most once per run.** Pinned by rows 1 and 2, which expect exactly one read at each span commit.
12. **Widening doubles the lookback (`batten.go:79` `b.walked = …`, `batten.go:104` `b.lookback *= 2`).** None. Both mutation survivors fall through to the whole-history fallback, and row 2 still passes.
13. **Whole-history fallback when a doubled walk adds no commit.** Weakly pinned: a run with no seed under a non-zero Since (the `TestComputeTiming` rows at `Since: at(0)`) would loop without it. No row asserts it.
14. **Widening stops at the earlier tag (`earlier != nil && !b.floor.After(*earlier)`, `batten.go` in `snapshotsOfRun`).** None. Row 1's floor lies after `earlier`, and the "never seeded" row in `TestCompute_SelectsRuns` (`runs_test.go:75`) runs with a zero Since.
15. **Span end at the next fresh seed via `isFreshSeed`.** Pinned by existing `TestCompute_SelectsRuns` rows (`runs_test.go:60-80`).
16. **Landed and in-flight timing runs share one index.** Pinned by `TestComputeTiming_WalksTheRecordsBranchOnce`, which expects 1 walk with a PathPrefix.
17. **Signature moves in `Compute`/`selectRuns`/`selectInFlight`.** Pinned through every existing `Compute`/`ComputeTiming` test.

**Findings**

1. **MEDIUM · design · `internal/reportengine/batten.go:104` (and `:79`)**
   - Summary: no test tells a doubled walk from a jump to the whole-history walk, so the bound the widening exists for is unpinned.
   - Evidence: mutation survivors at `batten.go:79` and `batten.go:104`. In `runs_test.go:255-259`, with either mutant, row 2 still reads `l0` and `l270` once each, because `widen` falls through to the full walk (`batten.go:120-125`).
   - Fix: in that row, assert the records walks' `Since` sequence from `w.repo.logQueries`: `at(82)`, then `at(-86)`, and no zero-Since walk.
2. **MEDIUM · design · `internal/reportengine/landings.go:149-154`**
   - Summary: no row opens a time-join window before `landingWalkStart`, so dropping the floor-lowering loop goes unseen.
   - Effect if it broke: a done mark between the checkpoint and the walk start would read at the wrong time.
   - Evidence: every `TestCompute_Landings` row uses `Since: at(-500), FreshDays: 3` (`landings_test.go:52`), so the floor is `at(-572)`. All checkpoints are at hour ≥ 20 (`landings_test.go:163-341`), and the new test's `cp` (85) lies after its floor (56).
3. **MEDIUM · design · `internal/reportengine/boardhistory.go:46-53`**
   - Summary: the `before = tip` branch has no row. That is the case where no board commit falls inside the window, as on a quiet board.
   - Effect if it broke: every landing would lose its labels (kind `other` plus a "no board commit holds an entry" finding).
   - Evidence: in every windowed test, board commits lie after the floor (`landings_test.go:123-377`, `472-478`; `weeks_test.go:112`, `215` with `Since: at(-500)`).
4. **LOW · design · `internal/reportengine/batten.go` `snapshotsOfRun` loop break on `earlier`**
   - Summary: no row has a reused slug's run with no seed after its earlier tag under a non-zero window. Removing the stop would widen to the whole history unseen, since the result is the same and only the walk count changes.
   - Evidence: `runs_test.go:240-250` finds its seed with the floor after `earlier`. `runs_test.go:75-80` runs with a zero Since.

Tool calls:
- Grep of `Since:`, `w.checkpoint(` and `w.boardCommit(` in reportengine tests: the manifest lacked the windows and hours of the existing landing and weeks rows.
- Grep of `since:` and row names in `runs_test.go`, `cost_test.go` and `timing_test.go`: the manifest lacked which run-selection rows use a non-zero window.
