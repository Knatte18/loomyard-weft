# Decision record — ide-attach-task

## Goal

The seeded `reed attach` task in a worktree's `.vscode/tasks.json` runs on a key binding, `ctrl+alt+a`, that the seeded lyx block in the user's `keybindings.json` carries.

## Scope

- `internal/vscode`: the keybindings block gains one entry running the `reed attach` task through `workbench.action.tasks.runTask`, with the task's label as its argument.
- The task itself stays as it is: on demand, no `runOptions`, never run on folder open.
- The package doc, the file header of `keybindings.go` and `ideengine.Spawn`'s doc stop calling the block terminal-only.
- Out of scope: dropping the task or `tasks.json`, any change to the four Alt-arrow entries, the merge or write mechanics of `SeedKeybindings`.

## Decisions

### The binding is `ctrl+alt+a`

Decision: the entry binds `ctrl+alt+a`.
Rationale: the operator decision picks `ctrl+alt+a` unless a VS Code default takes it;
the default keybindings of VS Code 1.142, core and built-in extensions, bind nothing to `ctrl+alt+a`, so the fallback `ctrl+shift+alt+a` is not needed.

### The entry carries no `when` clause

Decision: the entry runs `workbench.action.tasks.runTask` with args `"reed attach"` and no `when` clause.
Rationale: VS Code has no context key for a task's presence, and the binding must work whether focus is in the editor or the terminal.
Bound: the binding is user-wide, so in a folder without the task VS Code shows its own task-not-found notice; it runs nothing else.

### One label constant feeds the task and the binding

Decision: the task's label is one package constant read by both `writeAttachTask` and the block entry.
Rationale: the binding finds the task by label, so the two must not drift.

### Block entries carry their command and raw args

Decision: each block entry names its key, command, raw JSON args and optional `when`, and `keybindingsBlock` renders every entry from those fields.
Rationale: the block now holds entries of two commands; the sendSequence entries keep their texts written unexpanded, as before.

## Constraints

- `PATTERN-comment-line-breaks`: Go comments one sentence per line.
- `PATTERN-test-economy`: the new behavior extends existing tests where one covers it.
- A changed block replaces an older lyx block in place through the existing merge, so existing users get the new entry on their next `lyx ide` spawn.

## Auto-mode assumptions

- No VS Code default binding on any platform takes `ctrl+alt+a`; checked against the installed VS Code 1.142 bundle and its built-in extensions' manifests.

## Open risks

- A third-party extension the operator installs may bind `ctrl+alt+a`; the user-level binding then wins over the extension's default, which is the intended precedence.

## Acceptance criteria

- The seeded block holds an entry binding `ctrl+alt+a` to `workbench.action.tasks.runTask` with args equal to the generated task's label.
- The generated `tasks.json` still holds the one task with no `runOptions` and no `dependsOn`.
- The four Alt-arrow sendSequence entries render unchanged.
- An older lyx block in a user's file is replaced in place by the new block, as the existing merge tests pin.
