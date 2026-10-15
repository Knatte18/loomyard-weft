# ide: VS Code seeding drops the folderOpen chain for an on-demand attach

Fixes #633.

Batten's Seed-IDE row wrote the interactive `folderOpen` chain (`reed up`, `reed add --if-absent --unless-name orch --cmd claude --name claude --focus`, `reed attach`) into a driven pair's `.vscode/tasks.json`.
Opening such a pair in VS Code then added a bare, idle `claude` strand to the pair's reed session.

## What changes

- `vscode.WriteConfig` writes a `tasks.json` holding a single `reed attach` task with the stamped absolute `lyx` path and no `runOptions`, so VS Code runs it only from "Tasks: Run Task" and runs nothing on folder open.
- `vscode.TaskChain`, `TaskChainInteractive`, `TaskChainAttachOnly` and the `claudePath` parameter are deleted; there is one tasks.json shape.
- `ideengine.Spawn` (`lyx ide spawn`) and `ideengine.SeedTask` (Seed-IDE) keep sharing one writer, renamed `writeWorktreeConfig`, which no longer resolves `claude`.
- tasks.json is still overwritten on every write, so an existing worktree holding the old chain is replaced on its next `lyx ide spawn` or Seed-IDE; settings.json, the info/exclude handling and the tracked-tasks.json skip are unchanged.
- Docs: the ide row in `docs/overview.md` and the operator-session paragraph in `internal/loomcli/doc.go` no longer describe the chain.

## Design decisions

- The whole `folderOpen` chain goes, for every caller, rather than switching Seed-IDE to the attach-only variant: runs are watched in their own terminal window, and `reed add claude` must never run from inside VS Code.
- An on-demand `reed attach` task stays as the VS Code shortcut for `lyx reed attach`; it needs no preceding `reed up`, since `reed attach` boots a cold worktree's session itself.
- No keybindings.json entry is added for the task; the seeded keybindings block stays dedicated to forwarding reed's tmux keys.
- reed's `--unless-name` and `--if-absent` flags on `reed add` stay as operator-facing flags, though no lyx-generated config uses them any more.

## Tests

- `internal/vscode/config_test.go` pins the single task's label, command, args, presentation, and absence of `runOptions` and `dependsOn`, plus the bare-name fallback, the settings.json keep and tasks.json overwrite, and the untouched `.gitignore`.
- The ideengine integration checks assert the attach-only shape after `Spawn` on task pairs and the prime, and now also after `SeedTask`, which is the Seed-IDE path the bug came through.

## Risk

A worktree seeded before this change keeps the old chain until it is re-seeded or re-spawned; opening it in VS Code before then still runs the old chain.
