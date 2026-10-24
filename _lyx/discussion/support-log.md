# Support log — ide-attach-task

## Interview

A darn run holds no interview: the writer read the board entry, including its operator decision of 2026-10-24, and decided the rest on its own judgment.

## Rejected alternatives

- Dropping the task and `tasks.json` from the seed: the operator decision keeps the task.
- `ctrl+shift+alt+a`: the fallback applies only when a VS Code default takes `ctrl+alt+a`, and none does.
- A `when` clause such as `terminalFocus`: it would stop the binding from working with focus in the editor, which is where a freshly opened window starts.
- Writing the binding into the workspace instead of the user file: VS Code reads key bindings only from the user-level `keybindings.json`.

## Review rounds

_No rounds yet._

## Question ledger

- Does a VS Code default take `ctrl+alt+a`? Resolved: no; the installed VS Code 1.142 workbench bundle holds no keybinding with that chord's keycode and no built-in extension manifest binds it.
- Should the binding be scoped with a `when` clause? Self-pick: no, for the reason in Rejected alternatives.
