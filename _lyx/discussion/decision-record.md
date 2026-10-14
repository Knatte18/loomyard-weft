# Decision record — seed-vscode-config

## Goal

Opening the run's terminal window, seeding the pair's VS Code config and opening VS Code become three shed producers, each its own row in batten's recipe and each switched by a `batten.yaml` flag.
A row whose flag is off passes straight on to the next row.

## Scope

- One switched-action producer in `internal/shedadapters`, registered in `internal/shedrecipe` under three engine names: `OpenWindow`, `SeedIDE`, `OpenIDE`.
  Each reads its flag and its action from a seam on `shedrecipe.Env`, so any recipe can take the rows.
- `internal/ideengine` gains a seed entry point (title-bar colour and `.vscode/` config with the folder-open task chain, info/exclude included, no user key bindings) and an open entry point (launch VS Code on the pair's folder); `lyx ide spawn` keeps its behaviour, built from the same pieces.
- A new per-worktree config module `batten` (`batten.yaml`), declared in `internal/battenshed` and registered in `configreg`, with `open_window` (true), `seed_ide` (true), `open_ide` (false), all open to the local layer.
- Batten's recipe gets three rows, `Open-Window`, `Seed-IDE`, `Open-IDE`, between `Seed-Child` and `Run-Shed`.
- `lyx batten run --window` still passes the hidden `--open-terminal`, which now forces `open_window` true for that run.
- Removed: the window opening inside the Worktree-Create seam and the InnerRun row, `InnerRunDeps.OpenTerminal`, `OpenTerminalOnce`, the terminal-opened once-marker and its clearing in arm.
- Docs: `internal/shedadapters`, `internal/battenshed`, `internal/battenrecipe` and `internal/ideengine` package docs, `docs/overview.md`'s batten entry, CLAUDE.md's "Watching runs".
- Fabric is not touched.

## Decisions

- **One producer type, three engine names.**
  Rationale: the three rows share one behaviour (read a flag, act, never halt the run on a failed act); three engine names keep each row independently placeable and its seam independently wired, without three copies of the same Call.
- **Rows sit after `Seed-Child`, not directly after `Worktree-Create`.**
  Rationale: a task's `run_settings` reach the pair's local config layer only in `Seed-Child`, and the scope requires them to set the flags per task; `Seed-Child` writes, commits and spawns a detached push, so the window still opens seconds after the pair exists and before the child's run starts.
- **Flags are read at row time from the task worktree's layered `batten.yaml`** through `configengine.LoadOrTemplate`.
  Rationale: the local layer is the pair's, and an absent file in an older hub resolves the template defaults.
- **A failed action warns and the row reports Done.**
  Rationale: a window, a VS Code config or a VS Code launch is an operator convenience; today's window open already only warns. Bound: the row skips nothing but its own action, and the next row runs as if the action succeeded.
- **A flag that cannot be read is a hard error of the row.**
  Rationale: an invalid `batten.yaml` is a config defect the operator must see; the error names the file, and `lyx batten run <slug>` resumes at the row once fixed.
- **`--open-terminal` overrides `open_window` to true only.**
  Bound: it opens a window the config would have skipped, and never changes the other two flags.
- **`seed_ide` writes what `lyx ide spawn` writes for a task pair minus the user-level key bindings, and `open_ide` launches only.**
  Rationale: the board entry keeps the key bindings with `lyx ide`; launching alone keeps the two flags independent.
- **Tests never open a desktop window or VS Code.**
  Rationale: battencli's seams for the window and the VS Code launch are injectable, and its test mains refuse them, as `spawnPairPush` already is.

## Constraints

- `PATTERN-told-geometry`: `shedadapters`, `shedrecipe` and `battenshed` take every path and seam from the caller; only battencli resolves locations.
- `PATTERN-config-strictness` and `PATTERN-config-layers`: the batten module loads through `configengine` alone, with one of the two load forms.
- `PATTERN-shed-recipe-registry`: the engines join the one registry map.
- `PATTERN-spawn-observability`: the window and VS Code launches log their spawn.
- `PATTERN-leaf-packages` and battenshed's import allowlist: `configengine` and `yaml.v3` join battenshed's allowlist as genuine dependencies.
- Row names are durable; the new rows join `battenrecipe`'s constants and coverage guard.

## Auto-mode assumptions

- Placing the rows after `Seed-Child` meets "the window still opens as soon as the pair exists" closely enough; the parent may move them.
- A run resumed past `Open-Window` with `--window` opens no window, since the InnerRun fallback is removed as the scope asks.
- `batten.yaml` is per-worktree, not hub-wide, since the flags are a task-level choice.

## Open risks

- A plain `lyx batten run` now opens a desktop window by default where today only `--window` did; a headless machine only logs that no launcher exists.
- Tooling outside battencli's tests that drives a real batten run past `Seed-Child` on a desktop machine opens a window unless its config sets `open_window: false`.

## Acceptance criteria

- A switched row whose flag is false reports Done without calling its action.
- A switched row whose flag is true calls its action once and reports Done, also when the action fails.
- A switched row whose flag read fails returns an error and calls no action.
- Batten's recipe runs Worktree-Create, Seed-Child, Open-Window, Seed-IDE, Open-IDE, Run-Shed, Land, Worktree-Teardown in that order.
- `batten.yaml` resolves `open_window: true`, `seed_ide: true`, `open_ide: false` from its template, and a task's `run_settings` may set each.
- `--open-terminal` makes `Open-Window` open the window even when `open_window` is false.
- The Seed-IDE seam writes the pair's `.vscode/` config and colour and excludes `.vscode` from git, leaving the pair clean.
- The Worktree-Create seam and the InnerRun row open no window.

## Notes for the plan writer

- Window opening today: `battenCLI.openTerminal` and `openTerminalOnceCreated` in `internal/battencli/wire.go`; `OpenTerminalOnce` in `internal/battenshed/innerrun.go`.
- `ideengine.Spawn` in `internal/ideengine/spawn.go` holds the config write and the launch.
