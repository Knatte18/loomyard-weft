# Card 5 — batten-wire-open

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli/wire_test.go`
- `internal/battencli/testmain_integration_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/ideengine#SpawnDriven`
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
- `testmain_integration_test.go`: before `m.Run()`, replace `ideengine.CodeLauncher` with a no-op returning nil for the whole integration binary, so the lifecycle and records integration tests, which stub `Spawn` but keep the real `OpenIDE`, never start a real `code` process.
  Those tests then run the real driven path against real pairs; their teardown must still find the pair clean.
  If one refuses on the new `.vscode/`, that is a defect in the exclude step to fix in `SpawnDriven`, not a reason to stub `OpenIDE` away.

Docs, in `docs/overview.md`:
- The **batten** entry: `Run-Shed` opens VS Code on the task worktree's anchor folder once per run, after the child's bootstrap first spawns successfully, with a `folderOpen` task that runs `lyx reed attach` into the child's own session;
  a once-marker in `Run-Shed`'s scratch directory keeps a later re-spawn (an approved `awaiting` resume, a running child with no confirmed spawn) from reopening a window the operator closed, and a lost marker (a resume on another machine) opens it again;
  a failed launch (no `code` on `PATH`, a headless machine) is a warning only and never changes the row's outcome.
- The **ide** entry: the driven-pair variant batten uses writes an attach-only `tasks.json` (no `reed up`, no `reed add claude`), overwriting an untracked one and leaving a tracked one alone with a warning.
  It keeps `settings.json` when present, and keeps `.vscode/` out of git through the repository's shared `info/exclude` at the anchor subpath rather than `.gitignore`.
  Plain `lyx ide spawn` is unchanged.
Follow the repo's semantic-line-break rule.
No `CONSTRAINTS.md` change: no new cross-cutting invariant.

**ImpactSummary:** One new closure in batten's wiring and two test-harness edits inside `battencli`; the recipe, its row names and the CLI surface are unchanged.
