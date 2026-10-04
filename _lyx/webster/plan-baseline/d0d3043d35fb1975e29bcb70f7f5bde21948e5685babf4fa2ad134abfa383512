# Card 22 — ide-spawn-warnings

**Edit:**
- `internal/ideengine/spawn.go`
- `internal/vscode/workspace.go`
- `internal/vscode/workspace_test.go`
- `internal/vscode/config.go`

**Create:**
- `internal/ideengine/spawn_test.go`

**Intent:** Close the two ide-side hub-workspace follow-ups from the PR #301 review (D13, second and fourth bullets).

Log level by spawn target:
- `resolveSpawnTarget` logs a prime-name resolution failure at WARN only when the spawn targets the prime.
  A task spawn logs it at debug, since only its title-bar color degrades.
- `SpawnDriven` always targets a task pair.
- For `Spawn`, decide whether the slug names the prime without the failed prime-name read where a structural check exists in `fabricengine`.
  When the target kind cannot be told, log WARN, so a prime spawn never loses its warning.
- Factor the decision into a small pure helper (target kind and error in, log level out), so it is unit-testable without a hub.

Relative settings:
- `internal/vscode` gains a pure helper that returns the keys of a settings document whose string value starts with `./` or `../`.
- `writePrimeWorkspace`, the caller of `vscode.BuildWorkspace`, logs one WARN per spawn naming those keys when the list is non-empty.
  The message states that the values now resolve against the `_launchers/` workspace file's directory.
- `internal/vscode`'s package doc (in `config.go`) states this limitation.
- No setting is rewritten.

Tests:
- `spawn_test.go` covers the log-level helper: prime target with an error is WARN, task target with an error is debug, unknown target with an error is WARN, and no error logs nothing.
- `workspace_test.go` covers the relative-key helper: `./x` and `../x` string values are named; absolute paths, plain strings, non-string values and nested objects' handling are pinned; and empty or absent settings yield none.
- The WARN call itself stays a one-line use of the helper's result.

**ImpactSummary:** Adds two pure helpers and changes one log level and one new warning in ide spawning; no file content written by spawn changes.
