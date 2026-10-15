---
verdict: APPROVED
---

## Alignment at pr-gate: orch-context-overview

The diff of pull request 639 at `69bf627e6` against `main` delivers the board entry's goal: the prime's overview window, `lyx batten overview`, draws one orch line with the context tokens against the soft and hard thresholds, the cycle phase and the cycle count, above the runs in flight.

### Against the decisions

- D1: the orch line lives in `internal/battencli` (`overvieworch.go`, `overview.go`) and reads through `orchcli.PrimePaths` and `orchengine`, edges `battencli` already had.
- D2: `orchengine.PeekState` decodes `Paths.StatePath` with `os.ReadFile` and `json.Unmarshal`, takes no lock and creates nothing; an absent file is not found and a decode failure is an error.
  `orchStateCache` applies the two-strike tolerance, including a repeat of the same failed modification time, and draws `orch  state unreadable` on the second failure.
- D3: `loadOverviewThresholds` calls `orchengine.LoadConfig(fabricengine.ConfigDirs(location), "orch")` once at verb start; a failure logs at Warn and draws `-` for both thresholds.
- D4: the line has the recorded form, tokens in rounded-down thousands with `k` and plain under 1000, `-` for an unknown reading, the persisted phase as is, `watcher down` on a non-empty `WatcherExit`, no line for an absent file, and no padding into the run columns.
  The run lines render as on `main`, `no runs in flight` included; the orch line also precedes the runs-listing failure line.
- Docs: the verb's help text, the `battencli` package doc and the `orchengine` package doc name the orch line, as the scope's In list asks.
- Out-of-scope items are untouched: `lyx orch status`, the watcher, the orch state's fields, the orch config, the status bar and `docs/overview.md`.

The acceptance criteria each have a test: the render table in `internal/battencli/overview_test.go`, `TestOverviewSource_DrawsTheOrchLine`, `TestLoadOverviewThresholds`, and `TestPeekState` in `internal/orchengine/state_test.go`, which checks that no directory or lock file is created and that an unknown key is tolerated.

### Carried-over risks

- `testtiming-keep-directive-detached` (Webster-Review, round 1): fixed in `3d1a6af63`; the diff shows the `//testtiming:keep` directive directly above `TestCycleRequest_WriteProbeClear`.
- `run-pane-timing` editing `internal/battencli/overview.go`: that run has landed on `main` as `b8f8550cf` and this branch merged it in (`69bf627e6`); the diff against `main` changes the run lines nowhere, so the stage numbers it added are kept.
- The other open risks (`orch-any-worktree`, a prime with no orch state, a watcher killed outright) are recorded limits of the design, not deviations.

### Overlap

- Peers from `lyx loom peers` (`role-language-skills` row 9, `glyph-audit-tool` row 12, `kickstart-per-card` row 7; own row 22): none lists changed paths, and none touches the overview, the orch state or the orch config.
  No overlap, so no landing order applies.
- Parent commits: `lyx loom peers` lists none past the merge base.
  The one related landed change, `run-pane-timing` (`b8f8550cf`), is already in this branch's history and changes the run lines' stage cells, a different behaviour from the orch line.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` raises `internal/battencli` from 81 to 83 and `internal/orchengine` from 143 to 144; this alone is not an overlap.

### Superseded

No board entry is made obsolete by this run.
