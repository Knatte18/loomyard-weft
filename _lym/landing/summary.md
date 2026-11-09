# Title every console window `<hub>: <P#>: <T#>: <slug>`

With several hubs open on one machine, the operator could not tell which window belonged to which hub and project.
VS Code's `window.title` was the upper-cased shortname and the slug, and a terminal window from `lym reed window` was titled with the bare slug or `T4: <slug>`.
Now every console window takes one title form, read from the board when the window opens:

| What | VS Code | Terminal window |
|---|---|---|
| Task in project P3 | `ly: P3: T7: gate-shard-slow-package` | same |
| Task with no project | `ly: T12: adopt-fix-root-t1` | same |
| Task under beam P2 | `mo: P2: T4: wellbore-build-cost` | same |
| Beam P2 itself | `mo: P2: beam` | `mo: P2: orch` |
| Prime | `ly: prime` | `ly: orch` |
| Loom status window | `ly: P3: T4: tapestry` | |

A part that does not exist is left out, and the shortname is lower case, as `lym creel hubs` prints it.
A workspace file keeps its suffix: `ly: loomyard workspace`, `ly: <beam> workspace`.

## What changed

- `internal/consoletitle` is a new stdlib-only leaf holding the one function that joins a title, `Title(shortname, projectHandle, ownHandle, name)`, plus `Workspace` and the `prime`, `beam` and `orch` labels.
  It is a leaf so `fabricengine`, which may not import the board, forms the workspace title with the same function.
  `PATTERN-leaf-packages` gains its section, and a leaf enforcement test pins it.
- `boardengine.IDEWindowTitle`, `TerminalWindowTitle` and `RecipeWindowTitle` resolve a worktree's parts.
  They read the hub shortname and the board at the call.
  `<P#>` is the handle of the nearest open beam above the entry (`ParentBeam`), else of its `project`.
  An unreadable board is logged at Warn and leaves out every handle; a title never fails its caller.
- The places that formed a console title now call it:
  - `ideengine.lookFor`, behind `lym ide seed` and `lym ide spawn`;
  - `fabricengine.workspaceWindowTitle`, for the hub and beam workspace files;
  - `lym reed window`'s default title, when no `--title` is given;
  - batten's `{title}` placeholder, which the default `open_window_command` already passes;
  - `loomcli`'s status window, through `statusWindowTitle`.
- The reed status-bar `terminal` buttons in the task, beam and prime groups drop `--title {slug}`, so `lym reed window` forms the title itself.
- `docs/overview.md`, the batten template comment and the package docs describe the new forms.

## Design decisions

- **Prime and orch are two windows of one worktree.**
  The VS Code window on the prime reads `ly: prime`, and the terminal window on the prime's reed session, where the orch runs, reads `ly: orch`.
  A beam's windows read `<P#>: beam` and `<P#>: orch` the same way.
  A task's two windows share one title.
- **The `runs` window keeps its name.**
  The `runs` strand is not a console of its own, so `boardengine.TitleWithHandle` still names it.
  The loom status window is a window of a task's or beam's own session, so `RecipeWindowTitle` names it with the console title, the recipe standing where the slug does.
- **`lym reed window` forms its own default title** instead of each caller passing one.
  It resolves the worktree that holds the told directory, then the prime name through `fabricengine.PrimeName`.
  A directory in no worktree keeps its base name, and a `--title` still wins.
  A hub that wrote its own `buttons` group into a layer keeps any `--title {slug}` it holds, and so does one whose `open_window_command` passes `{slug}` in place of `{title}`.
  No config value changes meaning.
- **No part of lym reads a title back.**
  Nothing reads `window.title`, a terminal window's title or the `{title}` value.
  `desktop.window_command` gets the window's directory, not its title.
  reed orders windows by anchor and order, not by title, so the status window's longer name changes no order.

## Tests

- `consoletitle`: `TestTitle` and `TestWorkspace` pin every form, the lowered shortname and the left-out parts.
- `boardengine`: `TestWindowTitles` pins the prime, a beam, and a task in a project, in no project and under a beam, and covers a worktree off the board, no shortname, an unreadable board and no board.
  `TestRecipeWindowTitle` pins the status window's title.
- `ideengine.TestLookFor` pins the new VS Code titles; its integration tests and `fabricengine`'s workspace integration test pin the lower-cased shortname.
- `loomcli`'s `TestStatusWindowTitle` pins the status window with the shortname and project handle.
- `reedcli`'s window test pins that a missing `--title` takes the formed title through the `windowTitle` seam.
  A tier 1 test runs `terminalWindowTitle` against a hub fixture and a directory in no worktree.
- `reedengine`'s config test pins the shipped buttons without `--title {slug}`.

## To check by hand

- Open a task's terminal from its `terminal` button and check the title reads `<shortname>: <P#>: <T#>: <slug>`.
- From the prime's `terminal` button, check it reads `<shortname>: orch`.
- Run `lym ide seed` in a beam and check its VS Code window reads `<shortname>: <P#>: beam`.
- Start a loom run and check its status window reads `<shortname>: <P#>: <T#>: <recipe>`; the status window is titled through the shared function, in `boardengine.RecipeWindowTitle`.
- Check that `reedcli.terminalWindowTitle` in `internal/reedcli/window.go` is covered by its tests for a hub worktree and for a directory in no worktree.
