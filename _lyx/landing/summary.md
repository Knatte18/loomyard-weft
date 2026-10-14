# Batten's terminal window and VS Code become switched shed rows

Opening the run's terminal window, writing the pair's VS Code config and opening VS Code are now three ordinary rows of batten's recipe, each switched by a `batten.yaml` flag.

## What changes

- **Switched producer.** `shedadapters.SwitchedProducer` runs one told action when its told flag is on and is Done either way.
  A failed action only warns, and an unreadable flag is a row error.
  `shedrecipe` registers it under three engines, `OpenWindow`, `SeedIDE` and `OpenIDE`, each reading its own `shedrecipe.Env` seam pair (`shedadapters.SwitchedAction`), so any recipe can take the rows and supply flags from its own config.
- **Batten recipe.** `Open-Window`, `Seed-IDE` and `Open-IDE` run between `Seed-Child` and `Run-Shed`.
- **`batten.yaml`.** A new per-worktree config module, declared in `internal/battenshed/config.go` and registered in `configreg`: `open_window: true`, `seed_ide: true`, `open_ide: false`.
  Every key is open to the local layer, so a task's `run_settings` sets them per task.
  It loads through `LoadOrTemplate`, so a pair without the file resolves the template.
- **Wiring.** `battenCLI.switchedActions` reads each flag from the task worktree's config stack when the row runs.
  `Open-Window` opens the same Konsole / Windows Terminal window as before.
  `Seed-IDE` calls the new `ideengine.SeedTask`, which writes the title-bar colour and the interactive `.vscode/` chain with its `info/exclude` line but leaves the user's key bindings alone.
  `Open-IDE` calls the new `ideengine.OpenTask`.
  `lyx ide spawn` is unchanged and shares the config write with them.
- **`--window`.** It still passes the hidden `--open-terminal`, which now turns `Open-Window` on for that run whatever `open_window` says.
- **Removed.** The window opening inside the Worktree-Create seam and the InnerRun row, `InnerRunDeps.OpenTerminal`, `battenshed.OpenTerminalOnce`, the terminal-opened once-marker, and the marker's clearing at a fresh batten seed.
- **Docs.** The `battenshed` and `shedadapters` package docs, `docs/overview.md`'s batten entry, the `lyx batten run` help, CLAUDE.md's "Watching runs", and `PATTERN-config-strictness`'s degrading set, now including `battenshed`.

## Design decisions

- **The rows follow `Seed-Child`, not `Worktree-Create`.**
  The board entry asked for them right after `Worktree-Create`, but a task's `run_settings` reach the pair's local layer only in `Seed-Child`, so a flag read any earlier could not be set per task.
  `Seed-Child` writes and commits the seed and only spawns its push, so the window still opens seconds after the pair exists and before the child's run is spawned.
- **One producer type under three engine names.** The three rows share one Call; separate engines keep each row independently placeable and wired.
- **A failed action never halts the run.** All three actions are operator conveniences, and the window open only warned before as well.
- **A flag read failure is a row error.** It names the task worktree's `batten.yaml` and `lyx batten run <slug>` as the way forward.
- **`open_ide` launches only.** It writes nothing, so `seed_ide` and `open_ide` stay independent.

## Behaviour to note

- A plain `lyx batten run` now opens a desktop window by default, where before only `--window` did.
  On a machine with no launcher, the row logs a warning and passes on.
- A run resumed past `Open-Window` opens no window, since the InnerRun fallback is gone.

## Tests

- `switched_test.go` covers the producer's verdicts.
- `entries_switched_test.go` and the conformance table cover the registry entries.
- The batten recipe tests pin the new row order.
- `wire_seams_test.go` pins the seams' absent-pair refusals and the `--open-terminal` override.
- The batten integration tests step a real pair through the three rows, assert the template flags, a window opened on the pair and a `.vscode/` written with the pair left clean, and assert the create row opens no window.
