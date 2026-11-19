```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. I checked what the diff's signature, output and role changes force outside the lines it touches.

**Per change, the set I found:**

- **`vscode.BuildWorkspace` gains a `members` parameter.**
  - Callers: `internal/fabricengine/hubworkspace.go:215` and `:238`, plus `internal/vscode/workspace_test.go:45` and `:86`.
  - All four are updated: covered.
- **`isRunAgentSession` becomes `IsRunAgentSession`.**
  - No leftover reference to the old name in any `.go` or `.md` file: covered.
- **`RoleBoardServer` is added under `HubOwner`.**
  - It is registered at `boardcli/serveopen.go:101`.
  - `runproc/end.go:100` (`subjectPassSpared`) correctly leaves it out, as D11 requires.
  - `runproc/doc.go` and the `end_test.go` subject test are updated.
  - `slackcli` and `slackengine` use only `RoleSlackRelay` and need no change: covered.
- **`Render` now returns `board-data.js` and `board.html`.**
  - Replay claims: `boardengine/replay.go` and the `doc.go` claims list are updated.
  - Replay test: `replay_test.go` has the new rows.
  - Manifest test: `render_test.go` checks the manifest.
  - The render header comment is updated.
  - `docs/overview.md` is updated.
  - The sync stages the whole Bolt tree, so it needs no change: covered.
- **`Outputs.Labels` is added.**
  - `TestOutputs` is updated: covered.
- **`rerender` `Short` text is reworded.**
  - A grep of `docs/*.md` for the verb found no description of it to update: covered.
- **Two new verbs (`open`, `serve`) in the board subtree.**
  - `boardcli/doc.go` is updated.
  - `docs/overview.md` lists `internal/boardserve`.
  - The generic reach list shows the help-schema and CLI-form tests reach both verbs: covered.
- **New `lymdirs.BoardServeDirName`.**
  - Only `boardserve.Paths` uses it, and `lymdirs` is its one definition site: covered.
  - `pattern/PATTERN-lym-files.md` lines 19, 25 and 78 name creel, hibernate and the pause record as never-tracked state examples.
  - I did not open the rest of that file, so I can't say whether it enumerates the new `board-serve` directory. If it lists the `.lym` subdirectories exhaustively, it needs the new one. Not confirmed.
- **`fabricengine.InstallWorkspaceContent` and the `WorkspaceContent` hook.**
  - `cmd/lym/main.go` installs it.
  - `prerunseams_test.go` requires the call.
  - `fabricengine/doc.go` is updated.
  - The `ideengine` integration tests install it.
  - The `fabricengine` integration tests cover no hook, hook arguments, and refused members: covered.
- **VS Code `tasks.json` now holds two tasks, and `keybindings.go` has two new keys.**
  - Tests: `assertWorktreeTasks` replaces `assertAttachTaskOnly` at every call site, and `config_test.go` and `keybindings_test.go` are updated.
  - Docs: `docs/overview.md` is updated.
  - `config.go` and `keybindings.go` header comments are updated: covered.
- **Declared PATTERN change (`PATTERN-daemon-wakeups`).**
  - `pattern/PATTERN-daemon-wakeups.md` carries the boardserve entry.
  - `cmd/lym/daemonwakeups_test.go` lists `internal/boardserve/run.go`: landed.
- **Commit trailers.**
  - Every commit message in the range (`git log`, read earlier) is a bare title with no `Pattern-Departure:` or `Loosened-Check:` trailer, so none needs checking: covered.

**Finding:** LOW, `consistency`.
- **Location:** `internal/boardengine/render.go:2-4` (my read of the diff shows only a header addition at `:10-11`; line 2 itself is unchanged).
- **Summary:** the opening sentence of the file header still reads "Those are README.md, a project-&lt;slug&gt;.md ... and a design-*.md" as the full list of render outputs.
- **Evidence:** the diff adds `board-data.js` and `board.html` only in later header lines. I did not read the lines between to confirm the first sentence reads as an exhaustive list.
- **Fix:** add the two files to that first sentence.

**Tool calls:**
1. A batch of greps for `BuildWorkspace` callers, `isRunAgentSession`, `RoleSlackRelay` and the README references in `boardengine`, `fabricengine` and `hubreconcile`. My context lacked callers outside the diff.
2. A batch of greps over `PATTERN-lym-files.md`, the docs for `rerender` and the key bindings, and the `RoleBoardServer` uses. My context lacked the unchanged files.
