# Console titles lead with the handle, hub and project in brackets

Console window titles were `<hub>: <P#>: <T#>: <slug>`, such as `ly: P1: T2: adopt-worker-record`.
The task handle, which the operator looks for first, sat third, and a task under root showed no project.
A title now reads `<lead> [<hub>:<project>] <name>`:

- A task: `T2 [ly:P1] adopt-worker-record`.
- A task under root names root's project: `T5 [ly:P0] machine-name-file`.
- An orch leads with `ORCH`: the prime's is `ORCH [ly:P0] root` and a beam's is `ORCH [ly:P1] adopt-takeover`.
  The last part names the project, not the worktree.

## Design

- `consoletitle.Title` takes the lead first and joins lead, bracketed place and name with spaces.
  An empty part is left out with its separator, and the brackets go when both hub and project are empty.
  `Workspace` titles stay `[<shortname>] <name> workspace` in this form.
- `consoletitle.OrchLead` (`ORCH`) replaces the `PrimeLabel`, `BeamLabel` and `OrchLabel` constants.
- `boardengine.IDEWindowTitle` and `boardengine.TerminalWindowTitle` are merged into `ConsoleWindowTitle`.
  Both windows now show the same title, so the VS Code and terminal windows of a worktree no longer differ.
- A task with no project falls back to root's `P0`.
  The prime is titled by the root project (`P0`, `root`), not by its worktree name.
- A slug missing from the board is titled without handles, instead of reading an empty entry.
- `RecipeWindowTitle` keeps the task lead and drops the lead for a beam: `T4 [ly:P3] tapestry`, `[ly:P3] tapestry`.
- The docs, doc comments, the `open_window_command` template comment and the `reed window` help text show the new forms.

Callers and tests are updated to the new names and forms.

## To check by hand

- The `reed window` help text in `internal/reedcli/window.go` was updated to the new forms;
  check that no old-form title (`<hub>: <P#>: ...`, `<hub>: orch`) remains in it or in the nearby docs.
- Open a task, a beam, the prime and a task under root, and check each console window reads as above.
  Nothing was run against live windows.
