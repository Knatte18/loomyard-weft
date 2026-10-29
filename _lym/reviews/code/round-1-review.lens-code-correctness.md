```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff. For each changed function or branch, the case I ran and whether it was correct.

**Per-function list**

- `vscode.WorkspaceSettings`, nil or empty settings: `holdsValue` is false, so `decoded` stays `{}` and only the look's keys come out, with the prime green on the title bar. Correct.
- `vscode.WorkspaceSettings`, comments-only settings: same path as nil. Correct.
- `vscode.WorkspaceSettings`, BOM plus JSONC with a trailing comma: the BOM is trimmed and `stripJSONC` decodes it. Correct.
- `vscode.WorkspaceSettings`, `null`: `json.Unmarshal` returns no error and sets the map to nil, so the `decoded == nil` branch returns the settings unchanged with a warning. Correct. The warning logs `error=nil`, which is cosmetic.
- `vscode.WorkspaceSettings`, array or truncated JSON: `Unmarshal` errors and the settings come back verbatim, with the BOM kept. `BuildWorkspace` strips the BOM later, so this is correct.
- `vscode.WorkspaceSettings`, `workbench.colorCustomizations` that is not an object: the type assertion gives a nil map, and indexing a nil map is safe. The project colour falls back to `PrimeColor`, and `applyLook` replaces the key with a fresh map. Correct.
- `vscode.WorkspaceSettings`, beam settings holding a beam title-bar colour: that colour is kept on the title bar and the activity bar gets `WorkspaceColor`. Correct.
- `vscode.WorkspaceSettings`, an unchanged regeneration: the prime's input bytes are the same each time and `MarshalIndent` sorts keys. The output is byte-stable, so `regenerateHubWorkspace`'s "rewrite only on change" still holds. Correct.
- `vscode.WorkspaceSettings`, numbers or `<>&` in values: numbers go through `float64`, which is lossy beyond 2^53, and `<>&` are escaped to `\u00xx`. This is semantically equal JSON for VS Code and is the same behaviour as `writeSettings`. Not a defect.
- `applyLook` (extracted from `writeSettings`): the refactor keeps the order and keys. `writeSettings` still compares and writes the bytes. Correct.
- `workspaceWindowTitle`: with a shortname it gives `UPPER: name workspace`, and without one `name workspace`. Correct.
- `regenerateHubWorkspace` hub path: the error from `WorkspaceSettings` is wrapped and returned before any write, so nothing is written on that path. Correct.
- `regenerateHubWorkspace` beam loop: the look is applied after the unreadable-settings fallback to the prime's settings, so both sources get the beam title. Correct.
- `ideengine.seed`, prime slug: it writes the config and seeds the keybindings, then regenerates the workspace file. A regeneration error returns the keybindings result beside the error, as `Spawn` did. Correct.
- `ideengine.seed`, task slug: `launchPath` is the bare folder and the workspace file is untouched. Correct.
- `ideengine.seed`, `writeWorktreeConfig` failure: it returns an empty `seedResult`, so keybindings are not seeded. This matches the old `Spawn` and the old `Seed`. Correct.
- `ideengine.Spawn`: it returns `result.keybindings, CodeLauncher(result.launchPath)` after a successful seed, which is the old behaviour. Correct.
- `ideengine.seed`, prime name unresolved (`primeResolved` false): the workspace regeneration is silently skipped for `Seed`, with only the log from `resolveSpawnTarget`. `Spawn` degrades the same way, so it is consistent with the record. Correct.
- `idecli` seed wrapper, error from `Seed`: the error branch prints `seed failed` and ignores `fields`. Correct.
- `idecli` open wrapper, nil fields: `maps.Copy` from a nil source is a no-op. Correct.
- `idecli` seed, success: the result is `{slug, keybindings}` and the test pins `outcome`. Correct.
- `SeedKeybindings`, concurrent callers: batten's `Seed-IDE` row now makes every task seed write the user's `keybindings.json`. The write goes through a temporary file named by pid and nanosecond time and then a rename, so no torn file is possible. Two parallel seeds can still each read the old content and the later rename wins, but both write the same merged block, so the result is idempotent. Correct.

No findings.

**Tool calls (1 of 6)**
- `grep` of `SeedKeybindings` in `internal/vscode/keybindings.go`: my context lacked its write and concurrency behaviour, which matters now that every batten task seed calls it.
