# Bind ctrl+alt+a to the seeded reed attach task

The seeded `reed attach` task in a worktree's `.vscode/tasks.json` was reachable only through "Tasks: Run Task".
The lyx block that `lyx ide spawn` seeds into the user's VS Code `keybindings.json` now carries a fifth entry: `ctrl+alt+a` runs `workbench.action.tasks.runTask` with the task's label as its argument.
The task itself is unchanged: no `runOptions`, no `dependsOn`, never run on folder open.
An existing lyx block is replaced in place by the existing merge, so current users get the new entry on their next `lyx ide` spawn.

## Design decisions

- **Key choice.** `ctrl+alt+a` is used as is; the fallback `ctrl+shift+alt+a` is not needed, because the default keybindings of the installed VS Code (core and built-in extensions) bind nothing to it.
  A third-party extension that binds the same key loses to the user-level binding.
- **No `when` clause.** VS Code has no context key for a task's presence, and the binding must work from both the editor and the terminal.
  The binding is user-wide, so in a folder without the task VS Code shows its own task-not-found notice and runs nothing else.
- **One label constant.** `attachTaskLabel` in `internal/vscode/config.go` feeds both `writeAttachTask` and the block entry, so the binding cannot drift from the task it looks up.
- **Block entries carry command and raw args.** `keybindingsBlockEntries` in `internal/vscode/keybindings.go` now holds key, command, raw JSON args and an optional `when`, and `keybindingsBlock` renders each entry from those fields.
  The four Alt-arrow `sendSequence` entries render byte for byte as before.
- **Docs.** The `internal/vscode` package doc, the `keybindings.go` header, `ideengine.Spawn`'s doc and the `ide` entry in `docs/overview.md` no longer call the block terminal-only.

## Tests

`TestKeybindingsBlock` pins the full rendered block, including the four unchanged Alt-arrow lines and the new entry.
The existing merge tests cover in-place replacement of an older block.

## To check by hand

Not run: pressing `ctrl+alt+a` in a real VS Code window with a seeded `keybindings.json`, in the editor and in the terminal, to see the `reed attach` task start.
