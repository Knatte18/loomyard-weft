```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that fails when it breaks.

**Per behaviour, the test that pins it**

1. A start row entered `running` straight after a running pre-Run-Shed row commits without a push.
   - Pinned by `TestNewCommitStatusSeam/StartRowsCommitWithoutPush` (`commitstatus_test.go:58-61`): commits 8, pushes 3.
   - Also pinned by `TestNewCommitStatusSeam/SkippedGatedRowsCommitWithoutPush` (`:62`).
   - Breaking the branch at `commitstatus.go:264` makes every pushless call push, so both rows fail.
2. Run-Shed counts as a start row (the `producer == battenrecipe.NameRunShed` arm, `commitstatus.go:246`).
   - Pinned by `StartRowsCommitWithoutPush` (Open-IDE → Run-Shed) and `SkippedGatedRowsCommitWithoutPush` (Seed-Child → Run-Shed).
   - Dropping the arm adds one push to each, so both rows fail.
3. Worktree-Create is never a start row (the `producer != battenrecipe.NameWorktreeCreate` exclusion, `commitstatus.go:246`): **none**.
   - Every row that calls Worktree-Create calls it first, on an empty marker, so the push comes from the missing marker, not from the exclusion.
   - Rows that start on Worktree-Create: `StartRowsCommitWithoutPush`, `SkippedGatedRowsCommitWithoutPush`, `HaltedStartRowPushes` (`:59`, `:62`, `:64`).
   - Removing the exclusion passes every row.
   - The test's doc comment at `commitstatus_test.go:32-33` still claims "Worktree-Create … push".
4. The marker's producer must be a row before Run-Shed (`slices.Contains(preRunRows, marker.Producer)`, `commitstatus.go:264`): **none**.
   - No row enters a start row while the marker holds Run-Shed or a later row.
   - In `StartRowsCommitWithoutPush`, Land and Worktree-Teardown already fail `isStartRow`, so removing the conjunct leaves every count unchanged.
5. Only an incoming `running` state skips the push (`st == running`).
   - Pinned by `HaltedStartRowPushes` (`:64`): Seed-Child failed after Seed-Child running pushes.
   - Without the check, pushes would be 1 against the expected 2.
6. A marker holding a halt or pause state pushes (`marker.State == running`).
   - Pinned by `ResumedStartRowPushes` (`:63`): Seed-Child running after Seed-Child paused pushes.
   - Without the check, pushes would be 1 against the expected 2.
7. A missing or corrupt marker pushes.
   - Pinned by `MissingMarkerCommitsOnce` and `CorruptMarkerCommitsOnce` (`:56-57`), which now assert `wantPushes: 1`.
8. The start core rewrites the marker after its pushless commit (`writeMarker` passed to `startCore`, `commitstatus.go:242`): **none**.
   - With no marker write, the marker stays at Worktree-Create running and every start row still meets the start condition.
   - Land still fails `isStartRow`, so `StartRowsCommitWithoutPush` still counts 8 commits and 3 pushes.
   - No row repeats a start-row pair after a pushless commit, so the no-op skip after the start core is unchecked.
   - Example: the repeated `{"Run-Shed","running"}` of a Run-Shed self-bounce.
9. The start core takes no gate admission (`commitstatus.go:237-242`): **none**.
   - The table's deps set no `Admit` (`commitstatus_test.go:80-84`), so passing `deps.Admit` to `startCore` passes every row.
10. The existing rows' push assertion, changed from `pushes != tt.wantCommits` to `pushes != tt.wantPushes`.
    - Every existing row sets `wantPushes` equal to its `wantCommits` (`:53-57`).
    - The check is as strict as at the base; nothing was loosened.
11. Mutation survivor `commitstatus.go:225` (negated `err != nil` on the marker write).
    - The line only decides whether to log a warning, a log field no test can reasonably kill.
    - No finding.
12. Takeover meeting the starting machine's next push: no test in the diff.
    - This is the existing `PushRun`/`HandOver` path, held by `TestPushRun_HandsOverOnATakeover`; the diff adds no takeover code.
    - The relabelling of the PATTERN doc's "Held by" belongs to the affected-set lens.

**Findings**

- **MEDIUM, design.** `internal/battencli/commitstatus.go:242`: nothing checks that the start core's commit rewrites the marker.
  - A start-row pair repeated after a pushless commit, such as a Run-Shed self-bounce, would commit again with every row still passing.
  - Fix: add a row such as `{Worktree-Create running}, {Seed-Child running}, {Seed-Child running}` with `wantCommits: 2, wantPushes: 1`.
  - Evidence: `commitstatus.go:238-242`, `commitstatus_test.go:58-61`.
- **MEDIUM, design.** `internal/battencli/commitstatus.go:246`: the Worktree-Create exclusion from `isStartRow` is uncovered.
  - Every row reaches Worktree-Create with an empty marker, so removing the exclusion passes every row.
  - The test doc's "Worktree-Create … push" therefore goes unchecked.
  - Fix: add a row whose marker already holds a running pre-Run-Shed row when Worktree-Create runs, for example `{Seed-Child running}, {Worktree-Create running}`, expecting 2 pushes.
  - Evidence: `commitstatus_test.go:32-33`, `:59`, `:62`, `:64`.
- **LOW, design.** `internal/battencli/commitstatus.go:264`: the `slices.Contains(preRunRows, marker.Producer)` conjunct is uncovered.
  - No row enters a start row from Run-Shed or a later row.
  - Fix: add a row such as `{Land running}, {Seed-Child running}` expecting 2 pushes, or `{Run-Shed paused}, {Run-Shed running}`.
  - Evidence: `commitstatus_test.go:58-64`.
- **LOW, design.** `internal/battencli/commitstatus.go:237-242`: the start core's "takes no gate admission" is untested.
  - The table's deps leave `Admit` nil, so wiring `deps.Admit` into the start core goes unnoticed.
  - Fix: give the table's deps an `Admit` counter and assert it stays 0 for pushless rows.
  - Evidence: `commitstatus_test.go:80-84`.

Tool calls: none.
