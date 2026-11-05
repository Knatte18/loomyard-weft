```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**
- Built-in `sketch` type leads `Config.Types`: `TestLoadConfig_ScalarKeysAndTemplateDefaults`, `TestLoadConfig` rows in `config_test.go` (via `sketchLabel`), and `stepLabelsMapShaped…` and `stepLabelsListShaped…` in `verbs_test.go` (via `sketchPair`).
- board.yaml lists `sketch` itself: `TestLoadConfig_SketchListedInBoardYAML` (position and description). A listed `sketch` with a different handle letter is not tested, but `LoadConfig` takes no letter from board.yaml, so there is nothing separate to pin.
- Handle-letter collision on `S`: the `"a type on the sketch letter"` row in `TestLoadConfig_LabelsRefused`.
- `MakeSketch` changes kind, labels, handle, status, recipe, `depends_on` and `run_settings`, and keeps the discussed flag and title: `TestMakeSketch/HeldTaskBecomesASketch`.
- `MakeSketch` leaves an absent or done entry alone: `TestMakeSketch/AbsentAndDoneEntriesStay`.
- `MakeSketch` refuses an entry another depends on: `TestMakeSketch/DependedOnEntryIsRefused`.
- Run lock lets a held task become a Sketch: `HeldTaskBecomesASketch`, which runs through `checkRunLocks` and would fail if `endsAsSketch` were removed.
- Run lock still refuses other scope edits on a held entry that becomes a Sketch: none.
- Replay of a Sketch write passes the run lock: none.
- Migration of `discussed` to a Sketch at the next write: `TestDiscussedTaskBecomesASketchOnTheNextWrite`.
- Migration skips a task another entry depends on: none.
- README `Sketches` heading, first among the Notes and ahead of Research, with the `Sketch` link and module labels: `TestRenderSketchSection`.
- `Discussed` layer, section and title span removed: the old tests are deleted. The absence is not asserted, and nothing needs it to be.
- `--text` listing puts Sketches last under `sketches:`: the sketches row in `TestRenderCompact` and `stepListShowsASketch`.
- Start guard stops refusing `discussed`: the `DiscussedTaskRefuses…` row is deleted, and `UnreleasedTaskWithARunStatusFileResumes` now uses draft.
- `finishesAsSketch` and `commitStatusDeps.Finish` set per recipe: `TestRecipeKeyedWiring`.
- Seam calls `Finish` once on `done`, retries after a failure, and stops after a success: `TestNewCommitStatusSeam_Finish`.
- A motif run's finish makes the entry a Sketch, and a recipe with Finalize leaves the last running status: `TestWiredCommitStatus_FinishedRunEntry`.
- `sketchFinish` fallback to `draft` when `MakeSketch` is refused: none.

**Findings**

1. MEDIUM, design. Location: `internal/loomcli/wiring.go:233-236`. The `sketchFinish` fallback (warn, then `SetRunStatus(slug, draft)`) has no test. The Goal and decision record want the entry left as a draft when the board refuses, and `summary.md` admits it is covered only by reading the code. Evidence: `commitstatus_integration_test.go` has only the motif and tapestry rows, and no row has a dependent entry. Add a row to `TestWiredCommitStatus_FinishedRunEntry` with a motif task another entry depends on, wanting kind task and status `draft`. A fake `Finish` in `wiring_commitstatus_test.go` cannot reach this branch.

2. MEDIUM, design. Location: `internal/boardengine/runlock.go:152`. Nothing tests the `endsAsSketch` exemption's boundary. `HeldTaskBecomesASketch` covers only a clean `MakeSketch`. The focus asks whether "other body or scope edits" are still refused, and `endsAsSketch` passes any change that ends as a status-less note carrying the sketch label. No row tries a held task that becomes a Sketch while its title or body also changes, or a held task that becomes a note without the sketch label, which must still be refused. Add a `checkRunLocks` or `UpsertTask` row for each, wanting `ErrRunLocked`.

3. LOW, scope. Location: `internal/boardengine/doc.go:251` (the Run lock section). The docs say "the board replay runs the same check, so a replayed Sketch write is not refused", and `summary.md` repeats it. No replay test covers a Sketch write on a held entry, and the diff touches no replay test. Add one to the replay tests.

4. LOW, design. Location: `internal/boardengine/sketch.go:30-62`. `HeldTaskBecomesASketch` does not assert that `isolated`, `profile`, `pull_request` or `notify_on_land` are cleared, or that project, brief, body, issues and dates are kept. `sketchOf` drops the first four and keeps the rest. A regression such as dropping `Project` would pass. Add the fields to the task setup and the assertions in the same subtest, not a new test.

5. LOW, design. Location: `internal/boardengine/sketch.go:79-87`. The migration's refusal branch has no test: a `discussed` task that another entry depends on keeps the status. `doc.go:237` documents it. Add a row to `TestDiscussedTaskBecomesASketchOnTheNextWrite`, as a subtest or a second task. `MakeSketch` being a no-op for an entry that is already a Sketch is untested too, which is a one-line addition to `AbsentAndDoneEntriesStay`.

No test-economy finding. `stepListShowsASketch` overlaps the `RenderCompact` row, but it alone pins the hub CLI accepting the built-in type through `board.yaml`'s absence.

**Tool calls**
1. `git diff 05f99437..HEAD -- '*_test.go'`: my context held only the non-test diff.
