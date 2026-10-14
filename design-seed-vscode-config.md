# Terminal window, VS Code seed and VS Code open as shed producers with config flags

`seed-vscode-config` · Task · enhancement, ide, batten, shed · running · Publish

## Goal (operator, 2026-10-14)

Opening the run's terminal window, seeding a pair's VS Code config and opening VS Code become three ordinary shed producers, each its own row in batten's recipe and each switched on or off by a config flag.
A producer whose flag is off passes straight on to the next row.

## Seen (2026-10-14)

- The Konsole window is not a producer today: `lyx batten run --window` passes `--open-terminal` to the batten process, and `openTerminal` in `internal/battencli/wire.go` is called from inside the Worktree-Create row as soon as the pair exists, else from Run-Shed after its spawn, under a once-marker.
- `lyx ide spawn` (`internal/ideengine/spawn.go`) assigns the worktree a title-bar colour, writes its `.vscode/` config (including the folder-open task chain, kept out of git through `info/exclude`), seeds the lyx terminal key bindings into the user's `keybindings.json`, then launches VS Code.
  Nothing writes the colour or `.vscode/` when a pair is made, so `code .` in a pair never opened through `lyx ide` gives none of it.

## Scope

1. Three producers in `internal/shedadapters`, usable by any recipe:
   - `OpenWindow`: opens the run's terminal window attached to the pair's reed session (today's `openTerminal` behaviour).
   - `SeedIDE`: writes the pair's colour and `.vscode/` config, as `lyx ide spawn` does; the user-level key bindings stay with `lyx ide`.
   - `OpenIDE`: opens VS Code on the pair, as `lyx ide spawn` does.
2. Batten's recipe gets them as rows right after Worktree-Create, so the window still opens as soon as the pair exists.
   The window opening sewn into Worktree-Create and Run-Shed, and its once-marker, are removed.
3. Flags in `batten.yaml`, one per row: `open_window` (default true), `seed_ide` (default true), `open_ide` (default false).
   They are open to the local layer, so a task's `run_settings` sets them per task.
   `lyx batten run --window` overrides `open_window` for that run, so today's command keeps working.
   A later recipe that takes these rows reads the same key names from its own module's config.
4. Fabric is not touched: fabric knows nothing about IDEs (operator).
5. Docs: `internal/shedadapters` and batten docs, `docs/overview.md` if the batten row list is shown there, and CLAUDE.md's "Watching runs" if its wording of `--window` changes.

## Decided (operator, 2026-10-14)

- Producers in shed, not a fabric step and not batten-only code.
- Flags live in the config of the module that owns the recipe (`batten.yaml` for batten), not a shed default.

## Orch call, not the operator's (2026-10-14)

`SeedIDE` writes what `lyx ide spawn` writes, the folder-open task chain included, so `code .` gives the same as `lyx ide spawn`; the operator may overrule.
