# Card 7 — vscode-tasks-and-keys

**Create:**
- `internal/vscode#boardOpenTaskLabel`
- `internal/vscode#beamStartTaskLabel`
- `internal/vscode#writeWorktreeTasks`
- `internal/ideengine#assertWorktreeTasks`

**Delete:**
- `internal/vscode#writeAttachTask`
- `internal/ideengine#assertAttachTaskOnly`

**Edit:**
- `internal/vscode#WriteConfig`
- `internal/vscode#keybindingsBlockEntries`
- `internal/vscode#TestWriteVSCodeConfigCreatesFilesWhenAbsent`
- `internal/vscode#TestWriteConfigRewritesOwnSettingsKeysAndOverwritesTasks`
- `internal/vscode#TestKeybindingsBlock`
- `internal/vscode#TestSeedKeybindings`
- `internal/ideengine#checkTaskPairStaysClean`
- `internal/ideengine#checkOverwritesTasksKeepsSettings`
- `internal/ideengine#checkTaskSlugOpensBareFolder`
- `internal/ideengine#TestSpawnAnchoredScenario`
- `internal/ideengine#checkPrimeWritesHubWorkspace`
- `internal/ideengine#checkBeamOpensBeamWorkspace`
- `docs/overview.md`

**Uses:**
- `internal/vscode#attachTaskLabel`
- `internal/ideengine#readTasks`

**Intent:**
Write the `board open` task into every worktree's `tasks.json` and the two keys into the user's keybindings block (D6).

- `writeWorktreeTasks` replaces `writeAttachTask`: it writes `tasks.json` holding the existing `reed attach` task, unchanged, and a `board open` task:
  `"label": "board open"`, `"type": "shell"`, `"command": <lymPath>`, `"args": ["board", "open"]`, no `runOptions`, and a presentation with `"reveal": "always"`, `"focus": false`, `"echo": true` and `"panel": "shared"`.
  The terminal so shows `open`'s refusals and a failed opener's error and comes forward on every press without taking focus.
  `WriteConfig` calls it; its doc comment and `config.go`'s file comment name both tasks.
  `lymPath` keeps `WriteConfig`'s bare-`lym` fallback, which `ideengine` feeds from its `LymBinary` seam as for `reed attach`.
  `beamStartTaskLabel` is declared here beside the other labels and is read by card 9; no worktree `tasks.json` carries it.
- `keybindingsBlockEntries` gains `alt+shift+0` running the task `beam start` and `alt+shift+9` running `board open`, both through `workbench.action.tasks.runTask` with the label; `keybindings.go`'s file comment names them.
  Lym's keys keep clear of `alt+shift+1` to `alt+shift+6`, which mill's own block holds.
- `docs/overview.md`'s VS Code passage, which says the generated `tasks.json` holds one `reed attach` task and the `lym:begin` block binds only `ctrl+alt+a` to a task, names the `board open` task and its two keys: `alt+shift+9` runs `board open` and `alt+shift+0` runs `beam start`.
- `assertWorktreeTasks` replaces `assertAttachTaskOnly`: the worktree's `tasks.json` holds exactly `reed attach` and `board open`, each with an absolute command and no `runOptions`, `board open`'s presentation has `reveal` `always` and `focus` false, and no task is labelled `beam start`.

Bound: each key runs only `workbench.action.tasks.runTask` with a fixed label.
VS Code runs the first task of that label it finds, so a window whose tracked `tasks.json` defines its own `board open` or `beam start` runs that task.
A window lacking the label opens the task picker filtered by that label, as D6's bound states.

Tests:
- `TestWriteVSCodeConfigCreatesFilesWhenAbsent` and `TestWriteConfigRewritesOwnSettingsKeysAndOverwritesTasks` (tier0) assert the two tasks and `board open`'s presentation.
- `TestKeybindingsBlock` (tier0) pins the block with the two new lines; `TestSeedKeybindings` asserts that a second seed leaves exactly one `alt+shift+0` and one `alt+shift+9` entry.
- The ide tier1 checks call `assertWorktreeTasks`: a task pair (`checkTaskPairStaysClean`, `checkOverwritesTasksKeepsSettings`, `checkTaskSlugOpensBareFolder`, `TestSpawnAnchoredScenario`), the prime (`checkPrimeWritesHubWorkspace`) and a beam (`checkBeamOpensBeamWorkspace`), so seeding each writes `reed attach` and `board open` and no `beam start` into its `tasks.json`.

**ImpactSummary:** internal/vscode's tasks.json writer and key block, read by ideengine's seed and spawn; the user keybindings block changes on the next seed, and the ide integration checks swap their tasks assertion.

**Commit:** 7: ide: board open task and the board and beam keys
