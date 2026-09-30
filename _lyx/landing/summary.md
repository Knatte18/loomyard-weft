# batten opens VS Code on the new pair with reed attached

When batten starts a task, a VS Code window now opens on the new task worktree with the child run's reed session attached in its terminal.
The operator watches the run without typing a command.
Before this, the only route was `lyx ide spawn`, whose interactive chain adds a second, bare claude strand beside the child's driver.

## What changes

- `battenshed.InnerRunDeps` gains an `OpenIDE` closure.
  `Run-Shed` calls it right after a successful spawn of the child's bootstrap (`lyx loom start --no-attach`), before the post-spawn status read.
  A nil closure defaults to a no-op.
- `internal/battencli/wire.go` wires the closure to a new `ideengine.SpawnDriven`, which opens `<hub>/<slug>/<AnchorRel>`.
  That is the directory the bootstrap runs in, so `reed attach` resolves the child's own session.
- `internal/vscode.WriteConfig` takes a `TaskChain` selector.
  The new attach-only chain writes one `lyx reed attach` task on `folderOpen`, with no `reed up` and no `reed add claude`.
  `ideengine.Spawn` passes the interactive chain, so plain `lyx ide spawn` is unchanged.
- `fabricengine` exports `ExcludeAnchoredDir` and `PathTracked`.
  The driven path uses them to keep `.vscode/` out of git without editing any tracked file.
- `docs/overview.md`: the batten and ide entries describe the open step.

## Design decisions

- **Once per run.**
  A marker in `Run-Shed`'s scratch directory (`-ide-opened`) is written after the open is attempted, whatever its outcome.
  The approval re-spawn and the running-without-confirmation re-spawn therefore do not reopen a window the operator may have closed.
  `Call` clears the marker when it finds no child status file, so a marker from an earlier run of the same slug never suppresses a later open.
  A lost marker (resume on another machine) reopens the window on the next successful spawn, which is intended.
- **Failure is a warning.**
  An error from the closure is logged through `logger.Warn` and does not change `Run-Shed`'s outcome, reason or bounce accounting.
  The lifecycle continues with no `code` on `PATH`.
- **No dependency change for `battenshed`.**
  It stays told and imports neither `ideengine` nor `lyxcwd`; the closure is built in `battencli`.
- **`tasks.json` handling.**
  An untracked `tasks.json` is overwritten, because a leftover interactive chain would reproduce the bare-strand bug.
  A tracked one is left untouched, with a warning naming the slug, and VS Code still opens.
  `settings.json` keeps write-only-when-absent.
- **No tracked file is modified.**
  The driven path does not touch `.gitignore`.
  It writes an anchored `.vscode/` line into the shared `info/exclude` under `fabricengine`'s existing exclude lock, and skips the write when git already ignores the path.
- **Driver-independent.**
  `go` and `llm` children get the same chain; nothing reads the seed's `driver` field.
- The batten recipe's row names and order are unchanged.

## Risks and things to check by hand

- Not run: a real VS Code launch. Tests stub `ideengine.CodeLauncher`. Run a batten task on a machine with `code` on `PATH` and confirm the window opens with `reed attach` running in its terminal.
- A Linux machine with `code` on `PATH` but no display starts the process successfully, so no warning is logged and nothing opens.
- The `info/exclude` entry is repo-wide: it hides an untracked `.vscode/` in every worktree of the repo.
- A run resumed from an older build has no marker, so it opens a window on its next successful spawn.
- If the child's driver dies before VS Code finishes starting, the attach boots an empty session.
