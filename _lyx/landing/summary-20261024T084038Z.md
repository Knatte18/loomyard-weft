# Seed ctrl+alt+a to run the reed attach task

The `reed attach` task that `vscode.WriteConfig` writes into a worktree's `.vscode/tasks.json` ran only from "Tasks: Run Task", since nothing bound it to a key.
The block that `SeedKeybindings` merges into the user's `keybindings.json` now holds a fifth entry:

```jsonc
{ "key": "ctrl+alt+a", "command": "workbench.action.tasks.runTask", "args": "reed attach" },
```

The task itself is unchanged: it still has no `runOptions` and never runs on folder open.
An older lyx block in a user's file is replaced in place by the existing merge, so the binding arrives with the next `lyx ide` spawn.

## Design decisions

- **`ctrl+alt+a`, not the fallback.**
  The operator chose `ctrl+alt+a` unless a VS Code default already uses it, and `ctrl+shift+alt+a` otherwise.
  Neither the VS Code 1.142 workbench bundle nor any built-in extension manifest binds that chord, so the entry uses `ctrl+alt+a`.
- **No `when` clause.**
  VS Code has no context key for whether a task exists, and the key should work with focus in the editor as well as the terminal.
  In a folder without the task, VS Code reports the task as missing and runs nothing.
- **One label constant.**
  `attachTaskLabel` feeds both the task's `label` and the binding's `args`, because the binding finds the task by its label.
- **Entries carry command and raw args.**
  `keybindingsBlockEntries` now names each entry's key, command, raw JSON args and optional `when`, and `keybindingsBlock` renders every entry from those fields.
  The Alt-arrow entries render byte-for-byte as before.

## Tests

`TestKeybindingsBlock` pins the exact rendered block, which covers both the unchanged Alt-arrow entries and the new task entry.
The existing merge and seed tests compare against `keybindingsBlock` and cover the new block without changes.

## Docs

The `vscode` package doc, the `keybindings.go` file header and `ideengine.Spawn`'s doc no longer describe the block as terminal-only.
