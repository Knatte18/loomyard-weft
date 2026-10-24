# The seeded reed attach task in tasks.json runs on a shortcut

`ide-attach-task` · Task · enhancement, ide · running · Darn-Write

## Seen (orch, 2026-10-23)

The IDE seeder writes a `.vscode/tasks.json` holding one task, `reed attach` (`writeAttachTask`, `internal/vscode/config.go`, since `6667f79d`).
It carries no `runOptions`, so it never runs on folder open; it runs only from "Tasks: Run Task".
No key binding runs it: the block `SeedKeybindings` writes into the user's `keybindings.json` holds only the four Alt-arrow terminal sequences reed's tmux bindings listen for.

## Operator view, not a decision (2026-10-23)

The task is probably not needed; if it stays, it must not run on its own but sit on a shortcut.

## Idea

Either drop the task and `tasks.json` from the seed, or add a key binding to the seeded `keybindings.json` block that runs it (`workbench.action.tasks.runTask` with the task's label).

## Operator decision (2026-10-24)

Keep the task and bind it to `ctrl+alt+a` in the seeded `keybindings.json` block, unless VS Code's default keybindings already take that key; then use `ctrl+shift+alt+a`.
The task still never runs on folder open.
