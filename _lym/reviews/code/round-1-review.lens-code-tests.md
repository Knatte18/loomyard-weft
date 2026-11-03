```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. `ResolveCommandDir` (empty, relative, `.`, absolute): pinned by `TestResolveCommandDir`, `internal/gateslot/config_test.go`.
2. `StatCommandDir` refusing a missing dir with the resolved path: pinned indirectly by `gateslot` `TestCheckDirs`, the `driftengine` capture and compare rows, and `driftengine` `TestCheckDirs`. A `dir` that exists but is a file has no row, so the `info.IsDir()` half of the condition is unpinned.
3. `gateslot.CheckDirs` for a tier, the build, and an existing or unset dir: pinned by `TestCheckDirs`, `internal/gateslot/config_test.go`. A `build: {}` with no command being skipped is exercised only incidentally.
4. `driftengine.CheckDirs` for the capture step: pinned by `TestCheckDirs`, `internal/driftengine/config_test.go`. The compare step has no row: every row sets `capture` only, so the `{stepCompare, cfg.Compare}` entry could be dropped and nothing would fail (finding F1).
5. `runStep` refusing a missing dir before the slot and the log, with key, path and way forward, for `RunCapture`: pinned by `TestRunCapture`, row `AMissingDirIsRefusedBeforeTheCommand`. It asserts the error text, `rec.Reason`, `rec.State`, no log file and `slotFree`.
6. The same refusal for `RunCompare`, with the compare-only way forward and no `lym drift capture`: pinned by the new `TestRunCompare` row, `internal/driftengine/compare_test.go`.
7. A relative `dir` joining the worktree root rather than the anchor, and an unset `dir` running in the anchor, in `RunCapture`: pinned by `TestRunCapture`, rows `ARelativeDirJoinsTheWorktreeRootNotTheAnchor` and `NoDirRunsInTheAnchor`.
8. Dir resolution in `RunCompare`: only the missing-dir row. No row shows a compare's relative `dir` running in `<root>/dir` under a subdirectory anchor. `runStep` is shared with capture, but `RunCompare` passing `WorktreeDir` into it has no pin (finding F2).
9. Dir resolution and refusal in `RunSideCapture`: none. It sets `WorktreeDir` and `Name` itself in `capture.go`. The side-capture test at `capture_test.go:326` uses an empty `Dir`, which resolves to the anchor whatever `WorktreeDir` holds. Dropping `WorktreeDir` from the `RunSideCapture` call fails no test (finding F2).
10. `LoadConfig` no longer defaulting `dir` to `.`: pinned by the updated `TestConfigSpec_SetNestedKeys`, `TestConfigSpec_SetWholeSteps` and the `TestLoadConfig` row "a declared capture with no dir loads with an empty dir".
11. Filling `WorktreeDir` and `AnchorDir` correctly in `driftcli` (three requests) and `loomcli/driftwiring.go` (two requests): none. A grep for `WorktreeDir` in test files finds it only in `driftengine/capture_test.go` and `driftengine/compare_test.go`, besides unrelated `fabricengine` and `reedcli` tests. A swapped or empty `WorktreeDir` in the production wiring would not fail any test (finding F2).
12. `configcli` `--set` refusing a missing `dir` in the prime through `checkHubModule` and `dirs.root`: pinned by three new `TestDispatchHubWideBoard` rows: drift missing, drift present, gate `build.dir` missing. They assert the file is restored and that the resolved path under `fx.layout.WorktreePath()` is named.
13. Hub-wide editor save (`copyStagedToHub`) now running `checkHubModule` with `CheckDirs`: none. The existing editor-save rows use `darn`, which sets no `CheckDirs`. No row saves a drift or gate file with a missing `dir`, and none shows a save allowed when the file already had the same dir failure. Dropping `CheckDirs` from `copyStagedToHub` would fail nothing (finding F3).
14. The before/after rule keeping a write allowed when the file already failed the same dir check: no dir-specific row. The existing "darn editor save over a file already failing…" row covers `CheckLoad` only.
15. The `newLoadFailure` signature refactor in `setChecked` and `editMachine`: pinned by existing `--set` and machine-layer editor-save rows, which are unchanged.
16. The batten and dobby notices rendering paths against the worktree root (`wire.go` `PairDir` seam and `dobbynotice.go` `PairDir`): none that distinguishes it (finding F4). `lifecycle_integration_test.go:1568-1584` asserts `PairDir() == taskLocation.AnchorPath()`. The code now returns `WorktreePath()`, so that assertion and its name are stale. It passes only because the hub there is anchored at the root, where the two are equal. No row uses a subdirectory anchor to pin the full `dotnet/dotnet` path in a notice.
17. The new `driftengine` doc and template text: static prose, exempt.

**Findings**

- **F1, MEDIUM, class `scope`.** `internal/driftengine/config.go:239` (`CheckDirs`, compare entry), test `internal/driftengine/config_test.go` `TestCheckDirs`. No row declares a `compare` step with a missing `dir`, so the compare half of the check is unpinned. Fix: add a row with `compare: {command: "c {baseline_dir}", dir: gone, timeout_min: 1}` that wants `drift config key "compare.dir"`.
- **F2, MEDIUM, class `scope`.** Three paths are unpinned, in the code at `internal/driftengine/capture.go` (`RunSideCapture`) and `internal/driftengine/compare.go` (`RunCompare`), and the wiring at `internal/driftcli/cli.go` and `internal/loomcli/driftwiring.go`.
  - `RunSideCapture` has no test that sets a relative `dir` under a subdirectory anchor.
  - `RunCompare` has no row showing that a relative `dir` resolves against `<root>`.
  - The `driftcli` and `loomcli` wiring of `WorktreeDir` has no test.
  - Fix: add a `TestRunCapture` or side-capture row and a `TestRunCompare` row with the anchor at `dotnet` and `dir: dotnet`, asserting `pwd -P`. Add a wiring-level assertion that the requests carry the worktree root.
- **F3, MEDIUM, class `scope`.** `internal/configcli/hubwide.go:450-460` (`copyStagedToHub`, now through `checkHubModule`). No test saves a drift or gate editor file with a missing `dir`, and none covers the already-failing-same-dir allowance. Fix: add `TestDispatchHubWideBoard` rows for a drift editor save of a missing `dir` (refused, staging kept, hub file unchanged) and of a file whose `dir` already failed (written).
- **F4, MEDIUM, class `consistency`.** `internal/battencli/lifecycle_integration_test.go:1568-1584` still pins `InnerRun.PairDir` to `AnchorPath()`, while `wire.go` and `dobbynotice.go` now return `WorktreePath()`. The test cannot tell the two apart at a root anchor, so the notice change is unpinned. Fix: rename the step, assert `taskLocation.WorktreePath()`, and add a row, or a `readNoticeChild` unit, under a subdirectory anchor where the two differ.
- **F5, LOW, class `scope`.** `internal/gateslot/config.go:544` (`StatCommandDir`). No row covers a `dir` that resolves to an existing file rather than a directory. Fix: add a `gateslot` `TestCheckDirs` row whose `dir` is a regular file.

**Tool calls**
1. One `Bash` call with four greps: `WorktreeDir` in test files, notice and `PairDir` tests in `battencli`, side-capture tests in `driftengine`, and editor-save tests in `configcli`. My context did not hold the existing tests outside the diff.
