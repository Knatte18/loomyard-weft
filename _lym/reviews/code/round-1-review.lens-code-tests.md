```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the merge brings in or changes the test that pins it, or none.

**Per behaviour: pinning test, or none**

- **Pre-push guard on `Commit`, `Spawn` and `Push` in `commitStatusDepsGuardedBy` / `loomCommitStatusDeps` (`internal/loomcli/wiring.go`):** pinned by `TestPrePushCheckWiredIntoDeps` (`prepush_test.go:395-415`) and `TestPrePushCheckGuardedSeams` (`prepush_test.go:418`).
  - The wiring test checks that a stopped commit wraps `errPushStopped`, that a stopped spawn returns nil, and that a stopped push wraps `errPushStopped`.
  - It builds the deps through `commitStatusDepsGuardedBy` with an injected check, so the production `loomCommitStatusDeps(newPrePushCheck(...))` composition has no direct test. `wiring_commitstatus_integration_test.go:47` runs the real composition, but not a stopped path.
  - This is not a gap: the injected variant exists for this purpose and the one-line production call is trivial.
- **`newCommitStatusSeam` turns a commit stopped by `errPushStopped` into "no commit made" (`wiring.go:273-281`):** pinned by `commitrecords_test.go:92-95`, which drives `newCommitStatusSeam` with a `Commit` returning a wrapped `errPushStopped`.
  - It needs a manual check that no push starts and the transition goes on. I read only the stub setup, not the assertions beyond line 95.
- **Both sides' fields in `wire()` (the beam's `DefaultEngine` and assistant bounds beside main's guarded deps):** pinned by `TestWire_ReviewKeysReachTheEnv` (`wiring_test.go`, beam rows `defaultAssist`) and by the prepush tests above.
  - Nothing tests the two together in one wiring, and no new behaviour needs it.
- **`hibernate` module row in `configreg`:** pinned by `TestModules_SeedOnlyAndHubWideFlags` (`configreg_test.go:58`, `m.Name == "hibernate"` in `wantHubWide`).
- **`internal/hibernateconfig` as a degrading caller:** pinned by `configStrictnessDegradingSet` (`cmd/lym/configstrictness_test.go:61`).
- **`WaitNote.Steady` form and the change key in `status --watch`:**
  - `reviewstatus_test.go:191`, `:320` and `:395` pin `Steady` for each wait kind.
  - `shedverbs/status_test.go:316` and `:772-794` pin `PrintStatusLinesOnChange` and `statusWatchLine`.
  - `checklist_test.go:1283` pins `checklistPoll` carrying `Text` only.
- **Main's other behaviours:** each brought its own tests in main's commits, and the merge edits none of them.
  - The behaviours: `ParentNoticeFile`, child-recipe record, `ClearMotifRun`, dobby `drivable`, batten stray dir, fabric leftover refill, hibernate resume-on-wake, gatecli orphan pipe.
  - The tests: `prepush_test.go`, `childrecipe_test.go`, `paths_test.go`, `create_test.go`, `passes_test.go`, `cleanup_test.go`, `remove_leftoverdir_integration_test.go`, `hibernatecli/cli_test.go`, `hibernateengine/*_test.go`, `test_integration_test.go`. I confirmed these files exist in the range stat but did not read their assertions.
- **Merge-time edits to tests (`configreg_test.go`, `configstrictness_test.go`):** both are one-row additions with no duplicated coverage. `internal/configreg/configreg_test.go:88` already holds the beam's `codex` row.

**Findings:** none. I found no behaviour in the range without a pinning test, and no merge-time edit that duplicates existing coverage.

**Tool calls (2):**
1. A grep of the loomcli, shedverbs, cmd/lym and configreg tests for the guard, `WaitNote` and hibernate symbols, to find the pinning tests. My context held only the production code.
2. A grep for `newCommitStatusSeam` in loomcli tests, plus a read of `prepush_test.go:395-416`, to confirm the stopped-commit translation and the wiring test's assertions. I had not read those.
