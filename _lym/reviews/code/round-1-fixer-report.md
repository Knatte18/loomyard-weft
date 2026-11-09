```yaml
round: 1
fixed: [F1, F2, F3, F4, F5]
```

## Changes

- **F1** (commit `deb0d9d0d`, follow-up in the last commit): the loom status window is not the `runs` strand, so it now takes the console title.
  - `boardengine.RecipeWindowTitle(hubPath, slug, recipe)` is new in `internal/boardengine/handles.go`.
    It forms the worktree's console title with the recipe where the slug stands.
    `worktreeWindowTitle` gained a `name` parameter for that.
  - `statusWindowTitle` in `internal/loomcli/sharedbootstrap.go` calls it, so the window reads `ly: P3: T4: tapestry`.
    It passes no prime name, so it runs no git.
  - Tests: `TestStatusWindowTitle` gained a shortname-and-project row, and `TestRecipeWindowTitle` is new in `handles_test.go`.
  - Docs: `internal/loomcli/doc.go`, `internal/boardengine/doc.go`, both status-window lines of `docs/overview.md`, and the design decision and tests list of `_lym/landing/summary.md`.
  - The `runs` window in `internal/orchcli/runs.go` is left as it is, as the review said.
- **F2** (commit `0e4193b82`), fixed differently: `internal/reedcli/terminalwindowtitle_integration_test.go` is a tier 1 table test of `terminalWindowTitle` against a `hubforge` hub.
  Its two rows are a directory in no worktree, which takes its base name, and the prime worktree, which takes `<shortname>: orch`.
  The proposed `export_test.go` is not needed, since `window_test.go` is `package reedcli` and reaches the function directly.
  The Warn path of an unresolvable prime has no row, since a hub fixture resolves its prime.
- **F3** (commit `deb0d9d0d`): the doc comments of `IDEWindowTitle` and `TerminalWindowTitle` now say an unreadable board titles every worktree but the prime by its slug.
  `TestWindowTitles` gained the beam row `AnUnreadableBoardTitlesABeamBySlug`.
- **F4** (commit `deb0d9d0d`): `TestWindowTitles` gained a done-beam task, a task whose project is missing, and an empty `primeName` row, with the entries they need added to its board.
- **F5** (commit `64fb0f76a`): the test is now `TestCommandDeps_TitleIsTheTasksConsoleTitle`, its comments and the file header use the left-out-part wording, and it gained a shortname-and-project row.
  The comment of `commandDeps` in `internal/battencli/wire.go` already carried the left-out-part wording, so it needed no change.

Tier 0 passed for `boardengine`, `loomcli`, `battencli` and `reedcli`, and tier 1 passed for `reedcli`.
The last commit changed comments and docs only, and `boardengine` and `loomcli` were re-run at tier 0 after it.

## Gate repair

The gate found nine comment lines of this round that reached 120 columns or wrapped at a fixed column, so no test ran.
They are in `commanddeps_test.go`, `boardengine/doc.go`, `handles.go`, `handles_test.go`, `loomcli/doc.go`, `sharedbootstrap.go` and `terminalwindowtitle_integration_test.go`.
Each is now broken at sentence boundaries, one sentence per line, with no change of meaning.

A later gate run failed at tier 2: the loom smoke test `start beside a runs strand puts an own-window status strand into the overview window after it` in `internal/loomcli/smoke_drivingsurface_test.go` still expected the status window to be named the bare recipe.
F1 gives it the hub shortname, so the window reads `tst: tapestry`.
The assertion now expects `consoletitle.Title(hubforge.TestShortname, "", "", shedrun.RecipeTapestry)`.
This was the only test that read the window name from tmux.

## Disputed

- **Self-check, fix-code-correctness LOW**, ground `every fix worse`: the `NoPrimeNameTitlesThePrimesSlugAsATask` row cannot fail on the `primeName != ""` guard.
  It pins what F4 asked for, and the one input that reaches the guard is an empty slug, which a caller cannot hold.
  A row for it would test an input the code's own guarantees rule out.

## Deferred

Nothing deferred.

## Rulings

No ruling reached me.

## Self-check

- fix-code-correctness, LOW: the empty-`primeName` row cannot fail on the guard. Disputed.
- fix-consistency, LOW: `docs/overview.md` still said the status window opens with the run's board handle. Fixed.
- fix-consistency, LOW: the `boardengine` package doc lacked `RecipeWindowTitle`. Fixed, and its `TitleWithHandle` line now names the `runs` window.
- fix-proposal, LOW: `summary.md` said reed's window ordering leaves tmux window names alone. Fixed.
- fix-proposal, LOW: the `statusWindowTitle` comment claimed a run never starts in the prime. Fixed by dropping the claim.
- fix-proposal, LOW: a closed beam's handle shows as the `<P#>` in the done-beam row. No change, since it pins the behaviour F4 asked for.
- fix-affected-set: no findings.
