# Console titles put the handle first, with hub and project in brackets

Every console window was titled `<hub>: <P#>: <T#>: <slug>`, as in `ly: P1: T2: adopt-worker-record`.
The task handle, which the operator looks for first, sat third behind two colon-separated prefixes, and a task under root showed no project.
Titles now read `<lead> [<hub>:<project>] <name>`:

| Window | Before | After |
|---|---|---|
| Task in project P1 | `ly: P1: T2: adopt-worker-record` | `T2 [ly:P1] adopt-worker-record` |
| Task under root | `ly: T5: machine-name-file` | `T5 [ly:P0] machine-name-file` |
| Prime (VS Code / terminal) | `ly: prime` / `ly: orch` | `ORCH [ly:P0] root` |
| Beam P1 (VS Code / terminal) | `ly: P1: beam` / `ly: P1: orch` | `ORCH [ly:P1] adopt-takeover` |
| Run status window of a task | `ly: P3: T4: tapestry` | `T4 [ly:P3] tapestry` |
| Workspace file | `ly: loomyard workspace` | `[ly] loomyard workspace` |

A part that does not exist is left out, and the brackets go when both the shortname and the project handle are missing.

## What changed

- `consoletitle.Title(lead, shortname, projectHandle, name)` forms the new shape.
  `OrchLead` (`ORCH`) replaces the `prime`, `beam` and `orch` labels.
- `boardengine.ConsoleWindowTitle` replaces `IDEWindowTitle` and `TerminalWindowTitle`.
  The prime's and a beam's windows are the orch's in VS Code and in a terminal alike, so the two gave the same title.
  The prime names the root project, `P0` and `root`, from the board's `rootProject` and `RootSubject`, not the `prime` worktree.
  A task whose project chain ends at root takes `P0`.
- The callers in `ideengine`, `reedcli` and `battencli` call `ConsoleWindowTitle`.
- The package docs of `consoletitle`, `boardengine`, `fabricengine`, `battenshed` and `loomcli`, the batten template comment and `docs/overview.md` describe the new form.

## Design decisions

- A run's status window (`RecipeWindowTitle`) keeps the task's handle as its lead.
  In a beam it has no lead, `[ly:P2] tapestry`, since the window is the run's and not the orch's.
- A workspace file's title takes the bracket form, `[ly] loomyard workspace`, so every lym-titled window follows the same shape.
- A task whose `project` names an entry missing from the board leaves out the project handle, `T10 [ly] orphan`, instead of claiming root.
  A worktree off the board takes no lead, `[ly] stray`.
- The goal's note that the Slack channel names of `slack-per-orch-activation` follow the same rule is left to that board entry, which owns the Slack channel code.
- The `runs` window name (`TitleWithHandle`, `T4: <name>`) is a tmux window name, not a console window title, and is unchanged.
