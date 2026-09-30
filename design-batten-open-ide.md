Goal: when batten starts a task, a VS Code window opens on the new pair with the child run's reed session already attached in its terminal, so the operator watches the run without typing any command.

Batten owns the task worktree's lifecycle (`contracts/recipes/batten-recipe.yaml`: `Worktree-Create` -> `Seed-Child` -> `Run-Shed` -> `Worktree-Teardown`); opening the IDE is a lifecycle step, not part of the loom run.

Today:
- Nothing opens a window; the operator runs `cd <hub>/<slug> && lyx reed attach` by hand.
- `lyx ide spawn <slug>` (`internal/ideengine/spawn.go`, `internal/vscode/config.go`) writes `.vscode/settings.json` and `tasks.json` only when absent, and its `folderOpen` chain is `reed up` -> `reed add claude` -> `reed attach`.
  For a driven pair that chain is wrong: it adds a second, bare claude strand next to the ly-drive strand.

Wanted:
- A batten producer after the child's driver strand is up opens VS Code on the pair through `ideengine`.
- The generated `folderOpen` task for a driven pair runs `lyx reed attach` only — no `reed up`, no `reed add claude`.
- Launch failure (no `code` on PATH, headless machine) degrades to a warning; the lifecycle continues.

Open: where the strand-is-up point lies (batten's `Run-Shed` starts the child's driver, or `Seed-Child` does), and whether an existing `tasks.json` from a plain `ide spawn` is rewritten.