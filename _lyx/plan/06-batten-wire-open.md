# Card 6 — batten-wire-open

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli/wire_test.go`
- `internal/battencli/testmain_integration_test.go`
- `internal/battencli/records_integration_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `internal/ideengine#SpawnDriven`
- `internal/battenshed#InnerRunDeps`
- `internal/ideengine#CodeLauncher`

**Intent:**
Wire the open step into batten and document it, in one commit, since this is the card that makes the behaviour observable.

In `wire.go`, fill `InnerRunDeps.OpenIDE` with a closure that returns `ideengine.SpawnDriven(location, slug)`, passing the prime location `wire` already holds.
Before that call, confirm the pair is there with `taskWorktreeLocation(location, slug)` and return its error, so an absent pair is named in the warning rather than surfacing as a generic git failure.
Resolved on Call like every other seam here, never at wiring time.
`internal/battencli` is where `lyxcwd`-derived values live, so `internal/battenshed` stays told.
No new recipe row, config key or CLI flag.
The closure reads no seed field, leaving `internal/loomcli/bootstrap_test.go`'s driver-field tripwire untouched.

Tests:
- `wire_test.go`: add `OpenIDE` to `TestWire_LazySeams`'s non-nil table.
- `testmain_integration_test.go`: before `m.Run()`, replace `ideengine.CodeLauncher` with a no-op returning nil for the whole integration binary.
  The lifecycle and records integration tests stub `Spawn` but keep the real `OpenIDE`, so every one that reaches a successful spawn runs the real driven path against a real pair, and none starts a real `code` process.
  That covers `records_integration_test.go` through `callAwaiting`'s approval re-spawn, and the lifecycle tests whose status stub answers not-found through `Call`'s fresh spawn.
  A test whose status stub answers `done` before any spawn (`TestBattenIntegration_FourRowRun_SeedsChildCommitsAndTearsDown`) never opens.
- `records_integration_test.go`: an `OpenIDE` failure is only a warning, so a broken driven path would leave nothing behind and still tear down cleanly.
  The test therefore checks the driven path's effect directly.
  After the approved step (the one that spawns), it asserts that the pair's anchor `.vscode/tasks.json` (under `childLocation.AnchorPath()`) exists and decodes to exactly one task running `reed attach` on `folderOpen`,
  and that `git status --porcelain` in the pair's warp worktree is empty.
  Its existing assertions that the teardown step returns no error and the pair is gone afterwards stay as they are, and now show that removal succeeds with the driven `.vscode/` present.
- Every other existing `battencli` test stays unmodified and must pass.
  If one refuses on the new `.vscode/`, that is a defect in `SpawnDriven`'s exclude step to fix there, not a reason to stub `OpenIDE` away.

Docs, in `docs/overview.md`:
- The **batten** entry: `Run-Shed` opens VS Code on the task worktree's anchor folder once per run, after the child's bootstrap first spawns successfully, with a `folderOpen` task that runs `lyx reed attach` into the child's own session.
  A once-marker in `Run-Shed`'s scratch directory keeps a later re-spawn (an approved `awaiting` resume, a running child with no confirmed spawn) from reopening a window the operator closed, and a lost marker (a resume on another machine) opens it again.
  A failed launch (no `code` on `PATH`, a headless machine) is a warning only and never changes the row's outcome.
- The **ide** entry: the driven-pair variant batten uses writes an attach-only `tasks.json` (no `reed up`, no `reed add claude`), overwriting an untracked one and leaving a tracked one alone with a warning.
  It keeps `settings.json` when present, and keeps `.vscode/` out of git through the repository's shared `info/exclude` at the anchor subpath rather than `.gitignore`.
  Plain `lyx ide spawn` is unchanged.
Follow the repo's semantic-line-break rule.
No `CONSTRAINTS.md` change: no new cross-cutting invariant.

**ImpactSummary:** One new closure in batten's wiring and three test edits inside `battencli`; the recipe, its row names and the CLI surface are unchanged.
