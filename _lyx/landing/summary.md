# ide: seed one on-demand reed attach task, drop the folderOpen chain

Opening a lyx-seeded worktree in VS Code no longer starts anything.
Batten's Seed-IDE row wrote the interactive folderOpen chain (`reed up`, `reed add --if-absent --unless-name orch --cmd claude --name claude --focus`, `reed attach`) into a driven pair's `.vscode/tasks.json`.
Opening that pair in VS Code therefore added a stray bare `claude` strand to the running pair, with no prompt and no role.
Runs are watched in their own terminal window, and `reed add claude` must never run from inside VS Code, so the chain goes out of the seeding altogether.

Closes #633

## What changes

- `vscode.WriteConfig` writes a `tasks.json` with exactly one task, `reed attach`, running the absolute `lyx` path with `reed attach` in a new focused terminal.
  It has no `runOptions` and no `dependsOn`, so it is reachable from "Tasks: Run Task" and runs on demand only.
- `vscode.TaskChain`, `TaskChainInteractive`, `TaskChainAttachOnly` and the `claudePath` parameter are deleted.
  `WriteConfig` loses its `claudePath` and `chain` arguments; nothing in the config resolves `claude` any more.
- `ideengine.Spawn` (`lyx ide spawn`) and `ideengine.SeedTask` (batten's Seed-IDE) still share one config writer, renamed `writeWorktreeConfig` since the chain it was named for is gone.
  It stamps only the absolute `lyx` path; an empty path still falls back to the bare name `lyx`.
- `docs/overview.md`'s ide row, `internal/loomcli/doc.go` and the `internal/vscode` and `internal/ideengine` comments no longer describe a folderOpen chain.
- Tests pin the new shape in place (`internal/vscode/config_test.go` and the `ideengine` spawn integration tests) rather than adding parallel ones.

## Design decisions

- `tasks.json` is still regenerated on every write, overwriting an untracked file; a tracked `tasks.json` is still left alone with a warning.
  The overwrite is what replaces the old chain in existing worktrees.
- A worktree that already holds the old chain keeps it until its next `lyx ide spawn` or Seed-IDE, so opening it in VS Code before then still runs the old chain.
- The on-demand task has no preceding `reed up` task, since `reed attach` boots a cold worktree's session itself.
- No keybindings.json entry is added for the task.
- Reed's `add` verb and its `--unless-name` and `--if-absent` flags are untouched; they stay as operator-facing flags, now with no lyx-generated caller.

## To check by hand

- Not run by hand: opening a freshly seeded worktree in VS Code to confirm nothing starts, and running "Tasks: Run Task" → `reed attach` to confirm it attaches.
