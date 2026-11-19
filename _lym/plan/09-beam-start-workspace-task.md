# Card 9 — beam-start-workspace-task

**Create:**
- `internal/vscode#WorkspaceTasks`
- `internal/ideengine#WorkspaceContent`
- `internal/vscode#TestWorkspaceTasks`
- `internal/ideengine#TestWorkspaceContent`

**Edit:**
- `cmd/lym#newRoot`
- `cmd/lym#TestPersistentPreRunE_InstallsSeams`
- `internal/ideengine#checkPrimeWritesHubWorkspace`
- `internal/ideengine#checkBeamOpensBeamWorkspace`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/vscode#beamStartTaskLabel`
- `internal/fabricengine#InstallWorkspaceContent`
- `internal/ideengine#LymBinary`
- `internal/boardengine#RootSubject`

**Intent:**
Write the `beam start` task into the workspace files' `tasks` key through the card-8 seam (D6).

- `vscode.WorkspaceTasks(lymPath, beam, cwd)` returns the member name `tasks` and its value: `{"version":"2.0.0","tasks":[...]}` holding one `process` task, since a workspace file admits only `shell` and `process` tasks:
  `"label": "beam start"`, `"type": "process"`, `"command": <lymPath>`, `"args": ["beam", "start", <beam>]`, `"options": {"cwd": <cwd>}`, presented like `reed attach`: `"reveal": "always"`, `"panel": "new"`, `"echo": true`, `"focus": true`.
  An empty `lymPath` becomes `lym`, as `WriteConfig` does.
- `ideengine.WorkspaceContent` is the installed hook: it resolves `LymBinary()`, using `""` (so `lym`) on its error, and maps an empty beam to `boardengine.RootSubject` (`root`), so the hub's file runs `lym beam start root` in the prime's anchor and a beam's file runs `lym beam start <beam>` in that beam's anchor.
- `newRoot`'s PersistentPreRunE installs it with `fabricengine.InstallWorkspaceContent(ideengine.WorkspaceContent)` beside the pair folder namer, inside the same non-test block;
  `TestPersistentPreRunE_InstallsSeams` (tier0) requires the call.
- The regeneration's callers (`ide seed` and `spawn`, the fabric verbs, beam and adopt paths) all write the task through the one seam.
- `docs/overview.md`'s passage on the hub's and each beam's workspace file gains a line: each carries one `beam start` task, `lym beam start root` for the hub's and `lym beam start <beam>` for a beam's, which `alt+shift+0` runs.

Bound: the hook adds one `process` task running the lym binary, or bare `lym`, with the fixed args `beam start <beam>` in that worktree's anchor.
Fabric reads only the member names and refuses its own mark, and `vscode.BuildWorkspace` refuses `folders` and `settings` (card 8).

Tests:
- `TestWorkspaceTasks`, tier0 in `internal/vscode`, decodes the member: name `tasks`, one `process` task labelled `beam start` with the given command, `beam start <beam>` args, the given `cwd` and `reed attach`'s presentation; an empty `lymPath` gives the command `lym`.
- `TestWorkspaceContent`, tier0 in `internal/ideengine`, not parallel since it swaps the package's `LymBinary` seam and restores it: an empty beam gives `beam start root` in the given anchor, a beam slug `beam start <beam>`, and a `LymBinary` that errors gives the command `lym`.
- `checkPrimeWritesHubWorkspace` and `checkBeamOpensBeamWorkspace` (tier1) install `WorkspaceContent` with `InstallWorkspaceContent` and restore it in `t.Cleanup`;
they assert the hub workspace file holds one `beam start` task of type `process` running the stand-in `lym` with `beam start root` in the prime's anchor,
and the beam's file one running `beam start <beam>` in the beam's anchor.

**ImpactSummary:** cmd/lym installs one more process-wide hook; every hub workspace regeneration in a lym process now writes a tasks key; the two ide hub checks assert it.

**Commit:** 9: ide: beam start task in the hub and beam workspace files
