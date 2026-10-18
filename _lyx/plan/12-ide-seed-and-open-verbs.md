# Card 12 — ide-seed-and-open-verbs

**Edit:**
- `internal/idecli#Command`
- `internal/idecli#TestRunCLI_SpawnScenario`
- `cmd/lyx#excludedModules`

**Uses:**
- `internal/hubforge#AddPairWith`
- `internal/ideengine#SeedTask`
- `internal/ideengine#OpenTask`

**Intent:** Give the Seed-IDE and Open-IDE actions `lyx` verbs, so batten's template can spell their default commands (D4: "Every action that is Go code today gets a `lyx` verb").

`Command` gains two operator-audience subcommands under `lyx ide`, resolving the location in the existing pre-run:
- `seed <slug>`: `ideengine.SeedTask(l, slug)`, writing the task worktree's `.vscode/` config and title-bar colour and launching nothing; envelope `{"slug": <slug>}`.
- `open <slug>`: `ideengine.OpenTask(l, slug)`, starting VS Code on the task worktree's folder through `ideengine.CodeLauncher`, which detaches the editor and returns once it has started, so the window outlives the caller; envelope `{"slug": <slug>}`.
A missing slug is the usage error `spawn` gives, and an engine error is an error envelope naming the verb.
The module's no-arg listing and `spawn` and `menu` are unchanged, and the package comment names the four subcommands.
The `ide` entry of `excludedModules` keeps the module excluded from the sandbox suite (PATTERN-sandbox-coverage) and its reason names `seed` and `open` beside `spawn` and `menu`: `open` and `spawn` start a real VS Code window, `seed` writes into a task worktree a scenario would have to create a pair for, and `menu` is an interactive picker.

`TestRunCLI_SpawnScenario` gains steps over its hub, which first creates a real `child` pair with `hubforge.AddPairWith(t, h, "child", fabricengine.AddOptions{SkipPush: true})`, and whose `CodeLauncher` stub becomes a recorder of the directories it is called with:
- `seed child` exits 0 and writes `.vscode/settings.json` and `.vscode/tasks.json` under `h.PairCodeWorktree("child")` with no launch recorded;
- `open child` exits 0 and the recorder holds exactly one launch, on that path;
- `seed` with no slug exits non-zero with the usage text;
- `seed nosuch`, a slug with no pair, exits non-zero with an error envelope naming `seed`; and `open child` with the recorder returning an error exits non-zero with an envelope naming `open`.

**ImpactSummary:** Two new subcommands on the `ide` tree; existing verbs and every caller of `ideengine` are unchanged.

**Commit:** 12: lyx ide seed and lyx ide open
