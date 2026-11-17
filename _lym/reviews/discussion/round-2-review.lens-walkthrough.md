```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran one concrete case through each decision's mechanism and checked the end state against the decision's stated purpose.

**Walkthrough list**
- **D1, case: a body holding `&lt;/script&gt;` and `&lt;b&gt;`.** Purpose met. Go's default HTML escaping reaches the JSON, so no literal tag ends the script. The README already prints a render time (`internal/boardengine/render.go:213`), so the projection's render time adds no new commit churn.
- **D2, case: render on an unchanged binary.** Purpose met. The embedded bytes come back verbatim, and an identical rewrite leaves no git change.
- **D3, case: board write → `RenderToDisk` → manifest → sync → replay claim.** Purpose met. The manifest sidecar is the only gitignored render artefact (`render.go:13`, `render.go:33`), so both files are committed.
- **D4, case: an entry with a decision file, and a body link `[x](JaVaScRiPt:alert(1))`.** Purpose met. The decision file sits at the Bolt root (`render.go:71`), so the relative link from `board.html` resolves. The mixed-case scheme is neither `http` nor `https`, so it renders as plain text.
- **D5, case 1: the `board open` task from the prime window.** Purpose met. The task's cwd is the anchor folder that holds `.vscode`, the board `PersistentPreRunE` resolves it, `BoardDir(HubPath)` gives `board.html`, and the opener runs.
- **D5, case 2: a terminal in a subdirectory.** Purpose met through `lymcwd.ResolveWithin`.
- **D5, case 3: the verb under `--board-path`.** Broken; see F1.
- **D6, case 1: the hub window, key `alt+shift+0`.** Purpose met. The regeneration hands the hook `(hub, "", prime anchor)`, the hook returns a `tasks` member running `lym beam start root`, and `root` is a valid slug of `beam start` (`internal/beamcli/cli.go:457`, `internal/beamcli/target.go:16`). The workspace-level task runs with the prime anchor as cwd.
- **D6, case 2: a beam's window.** Purpose met. The beam's file gets `lym beam start &lt;beam&gt;`, and the beam's anchor is already read through `lymcwd.NamedWorktree` (`internal/fabricengine/hubworkspace.go`, `beamWorkspaceSettings`).
- **D6, case 3: `alt+shift+9` in a multi-root window.** Purpose met. Every folder's `tasks.json` copy runs the same verb, so the result is the same whichever copy runs.
- **D6, case 4: a seed of a task pair.** Purpose met. `ideengine.LymBinary` → `vscode.WriteConfig` writes `reed attach` and `board open` into the folder's `tasks.json` (`internal/ideengine/spawn.go:231`, `:265`).
- **D7, case: a project with sub-projects.** The mechanism holds: Tasks, Notes and Done under `Own entries` become `&lt;details&gt;` blocks. Whether the purpose still holds after the parent's phone decision is the handler's question, not this lens's.

**Findings**

F1
- severity: LOW
- class: consistency
- location: D5, first and second bullets, `_lym/discussion/decision-record.md:107-114`
- summary: Under `--board-path` there is no resolved layout, so the refusal has no anchored directory to name, and the bullet's "so" makes the flag a consequence of resolution when it bypasses it.
- issue: D5 says the verb sets the board directory from the resolved layout, "so the subtree's `--board-path` flag points the verb at that directory's `board.html`". It also says the refusal names "the worktree's anchored directory to run it from". With `--board-path` set, `PersistentPreRunE` skips resolution and `location` stays nil (`internal/boardcli/cli.go:56`, and the `*boardPathFlag != ""` branch at about `cli.go:135-136`). In that case the refusal has no anchored directory, and the record states no refusal text for it.
- fix: State that `--board-path` bypasses resolution and selects `&lt;board-path&gt;/board.html`. State what the refusal names when no worktree was resolved, for example `lym board rerender` alone.

Tool calls:
1. `internal/beamcli` `start` usage, `render.go` manifest and gitignore, and decision-link paths: checks D6's `beam start root` argument and D3/D4's file placement, which the manifest lacked.
2. `internal/beamcli/cli.go` `startCmd` and the root-slug constant: confirms `root` is a valid slug of `beam start`.
3. The `render.go` render-time line and the board critical section: checks whether D1's render time adds commit churn the README lacks today.
