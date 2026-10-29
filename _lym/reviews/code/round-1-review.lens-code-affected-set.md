```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for everything that calls, reads or asserts it, and for the docs and rules that describe it.

**Per change: the set found, covered or not**

1. `ideengine.Seed` signature (`error` to `(KeybindingsResult, error)`).
   - The only production caller is `internal/idecli/cli.go:106`, which the diff updates.
   - The test callers are `spawn_hub_integration_test.go` and `spawn_scenario_integration_test.go`, and both are updated.
   - Covered.
2. `slugVerbCommand` action signature.
   - Its callers are `seedCmd` and `openCmd`, and both are updated.
   - Covered.
3. `lym ide seed` JSON output gained `keybindings`.
   - `internal/idecli/cli_test.go` asserts it, and `docs/overview.md` states it.
   - Covered.
4. Seed now writes the user's `keybindings.json`, and the Seed-IDE row runs `lym ide seed`.
   - `docs/overview.md` (line 665) and `internal/battenshed/template.yaml` (the `seed_ide_command` comment) are updated.
   - `internal/battencli/cli.go:218` says only "Seed-IDE's writes the worktree's VS Code config", and `internal/battenshed/doc.go:82` says "writes the pair's VS Code colour and .vscode/ config". Neither contradicts the new behavior, since neither mentions key bindings.
   - Covered. Nothing here is wrong.
5. Workspace settings are no longer spliced verbatim.
   - `fabricengine/doc.go`, `overview.md` and `vscode/config.go`'s package doc are updated.
   - Not covered: `internal/vscode/config.go:18` (the Limitation paragraph) still says "BuildWorkspace splices the prime's settings verbatim into the hub workspace file". That is no longer true for the hub file, because the regeneration passes the settings through `WorkspaceSettings` first. `BuildWorkspace` itself still splices verbatim, so the claim is only partly wrong. See the finding below.
6. Prime seed regenerates the hub workspace file.
   - The `ideengine` package header comment, `overview.md` and the `idecli` header are updated.
   - Covered.
7. New `vscode.WorkspaceColor`, `WorkspaceSettings` and `applyLook`.
   - `WorkspaceSettings` is called from `fabricengine/hubworkspace.go` only, and `applyLook` is shared with `writeSettings`.
   - The tests cover them and the package doc is updated.
   - Covered.
8. Repository rules (`CLAUDE.md`, `PATTERN`).
   - The workspace file is still written only in `fabricengine`.
   - No PATTERN entry is declared, and none is needed.
   - No `Pattern-Departure` trailers appear in the commit messages.
   - No stencils are touched.
   - Covered.

**Finding**

- **LOW, consistency**: `internal/vscode/config.go:18`.
  - The package doc's Limitation says `BuildWorkspace` splices the prime's settings verbatim into the hub workspace file.
  - After this diff the hub file's settings come from `WorkspaceSettings`, which re-encodes them, so that sentence misdescribes the hub file.
  - The same doc was edited a few lines above (lines 11-12 and 15), so the edit missed this sentence.
  - The Limitation itself still holds, because a relative path still resolves against the hub directory.
  - Fix: reword it to "the hub workspace file's settings carry the prime's settings".

**Tool calls**
1. grep for the `Seed` and `slugVerbCommand` callers and for stale "verbatim" and "window.title" text across the Go, markdown and yaml files. I needed this because my context lacked the callers and the docs beyond the diff.
2. grep for `ide seed`, `Seed-IDE` and `seed_ide` mentions across the repository. I needed it to find the batten docs and help text beyond the diff.
3. `sed` of `internal/battencli/cli.go` lines 214-222, plus a grep for "verbatim" in the docs and `internal/vscode/*.go`. I needed it to read the batten help wording and find other "verbatim" claims.
