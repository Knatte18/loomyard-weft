# Batten: window, VS Code seed and VS Code open as switched shed rows

Opening the run's terminal window, seeding a pair's VS Code config and opening VS Code are now three ordinary shed producers.
Each is its own row in batten's recipe and is switched by a `batten.yaml` flag; a row whose flag is off reports Done and passes on to the next row.
Before, the window was opened from inside the Worktree-Create seam and Run-Shed under a once-marker, and nothing wrote the colour or `.vscode/` config when a pair was made, so `code .` in a pair never opened through `lyx ide` gave none of it.

## What changes

- `lyx batten run` now runs `Worktree-Create`, `Seed-Child`, `Open-Window`, `Seed-IDE`, `Open-IDE`, `Run-Shed`, `Land`, `Worktree-Teardown`.
- New per-worktree config module `batten` (`batten.yaml`, registered in `configreg`, declared in `internal/battenshed`) with `open_window` (default true), `seed_ide` (default true) and `open_ide` (default false).
  All three are open to the local layer, so a task's `run_settings` sets them per task.
- `lyx batten run --window` still works: its hidden `--open-terminal` now forces `open_window` true for that run, and touches no other flag.
- `Seed-IDE` writes the pair's title-bar colour and `.vscode/` config, folder-open task chain included, and excludes `.vscode` through `info/exclude`, leaving the pair clean.
  It does not touch the user-level `keybindings.json`; that stays with `lyx ide`.
- `Open-IDE` only launches VS Code on the pair, so the two flags stay independent.
- `internal/ideengine` gains a seed entry point and an open entry point; `lyx ide spawn` keeps its behaviour, built from the same pieces.
- Removed: the window opening in the Worktree-Create seam and the InnerRun row, `InnerRunDeps.OpenTerminal`, `OpenTerminalOnce`, and the terminal-opened once-marker with its clearing in arm.
- Docs updated: `internal/shedadapters`, `internal/battenshed`, `internal/battenrecipe` and `internal/ideengine` package docs, `docs/overview.md`'s batten entry, CLAUDE.md's "Watching runs", and `PATTERN-config-strictness`.
- Fabric is not touched.

## Design decisions

- **One switched-action producer, three engine names.**
  `OpenWindow`, `SeedIDE` and `OpenIDE` share one `Call` in `internal/shedadapters`, registered in `internal/shedrecipe`.
  Each reads its flag and its action from a seam on `shedrecipe.Env`, so any recipe can take the rows and reads the same key names from its own module's config.
- **The rows sit after `Seed-Child`, not directly after `Worktree-Create`.**
  A task's `run_settings` reach the pair's local config layer only in `Seed-Child`, and the flags must be settable per task.
  The window therefore opens seconds after the pair exists, before the child's run starts.
- **Flags are read at row time** from the task worktree's layered `batten.yaml` through `configengine.LoadOrTemplate`, so an older hub without the file resolves the template defaults.
- **A failed action only warns and the row reports Done**, as the window open already did.
  A flag that cannot be read is a hard error that names the file, calls no action, and lets `lyx batten run <slug>` resume at the row once fixed.
- **Tests never open a window or VS Code**: battencli's seams for both are injectable and its test mains refuse the real ones.
- `configengine` and `yaml.v3` join `internal/battenshed`'s import allowlist as genuine dependencies.

## Behaviour to be aware of

- A plain `lyx batten run` now opens a desktop window by default, where only `--window` did before.
  A headless machine only logs that no launcher exists; tooling that drives a real batten run past `Seed-Child` on a desktop machine needs `open_window: false`.
- A run resumed past `Open-Window` with `--window` opens no window, since the fallback in InnerRun is gone.

## To check by hand

Not run here: a real batten run on a desktop machine, to confirm the Konsole or Windows Terminal window opens after `Seed-Child`, that `seed_ide` leaves a coloured, git-clean pair for `code .`, and that `open_ide: true` launches VS Code.
