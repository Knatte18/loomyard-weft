Goal: when a loom run starts, a VS Code window opens on the new pair with the run's reed session already attached in its terminal, so the operator watches the run without typing any command.

Today:
- `lyx loom start --no-attach` spawns the ly-drive strand and opens nothing; the operator must run `cd <hub>/<slug> && lyx reed attach` by hand.
- `lyx ide spawn <slug>` (`internal/ideengine/spawn.go`, `internal/vscode/config.go`) writes `.vscode/settings.json` and `tasks.json` only when absent, and its `folderOpen` chain is `reed up` -> `reed add claude` -> `reed attach`.
  For a loom pair that chain is wrong: it adds a second, bare claude strand next to the ly-drive strand.

Wanted:
- `loom start` (or the orchestrator's start path) opens VS Code on the pair after the driver strand is up.
- The generated `folderOpen` task for a loom pair runs `lyx reed attach` only — no `reed up`, no `reed add claude`.
- `ide spawn` creates the pair when missing (ide depends on fabric, never the reverse).
- Launch failure (no `code` on PATH, headless machine) degrades to a warning; the run still starts.

Open: which verb owns the open (`loom start` flag vs. orchestrator calling `ide spawn`), and whether an existing `tasks.json` from a non-loom spawn is rewritten.