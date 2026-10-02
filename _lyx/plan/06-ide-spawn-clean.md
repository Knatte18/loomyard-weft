# Card 6 — ide-spawn-clean

**Edit:**
- `internal/vscode/config.go`
- `internal/vscode/config_test.go`
- `internal/ideengine/spawn.go`

**Delete:**
- `internal/ideengine/spawn_test.go`

**Create:**
- `internal/ideengine/spawn_clean_integration_test.go`

**Prosa:**
- `docs/overview.md`
- `CONSTRAINTS.md`

**Uses:**
- `internal/fabricengine#PathTracked`
- `internal/fabricengine#ExcludeAnchoredDir`
- `internal/agentname#RoleOrch`

**Intent:**
Make `lyx ide spawn` leave a worktree clean and reach existing primes with the new chain (D3, D4, D5, D6; fixes #334 and the folder-open half of #340).

`internal/vscode/config.go`, `WriteConfig`:

- The interactive chain's `reed add claude` row gains `--unless-name` with `agentname.RoleOrch`, so opening a prime whose orch strand is live attaches to the session without stacking `claude` below orch; `reed up` and `reed attach` stay as they are.
- The interactive `tasks.json` is written on every call, overwriting an existing file, as the attach-only chain already is; `settings.json` keeps write-only-when-absent.
- The `gitignore.Ensure` call and the `internal/gitignore` import go: neither chain touches `.gitignore`.
- The doc comment says both chains rewrite `tasks.json`, neither touches `.gitignore`, and the caller keeps `.vscode/` out of git.

`internal/ideengine/spawn.go`:

- Extract `SpawnDriven`'s tracked-check, exclude and write sequence into one unexported helper taking the chain and `claudePath`, and call it from both `Spawn` and `SpawnDriven`.
  The sequence: `fabricengine.PathTracked` on the anchor's `.vscode/tasks.json`; if tracked, one `logger.Warn` and no write of either file nor of `info/exclude`; otherwise `fabricengine.ExcludeAnchoredDir(worktreeDir, AnchorRel, ".vscode")`, logging when it changed the file, then `vscode.WriteConfig`.
- `Spawn` still launches after a tracked skip, the prime's hub workspace file included (`vscode.ReadSettings` already returns nil for an absent `settings.json`).
- Update the file and function doc comments: `Spawn` regenerates the interactive chain on every call and keeps `.vscode/` out of git through `info/exclude`.

What this lets through, and its bound: `Spawn` now overwrites an untracked `.vscode/tasks.json` at the spawned worktree's anchor, so operator edits to it are lost on the next spawn, the contract the hub workspace file already has.
A tracked `tasks.json` is never touched.
`Spawn` on a directory that is not a git checkout now fails at the tracked check, as `SpawnDriven` already does.

Tests:

- `internal/vscode/config_test.go`: the interactive-chain assertions expect `--unless-name orch` in the `reed add claude` args; `TestWriteVSCodeConfigDoesNotClobber` keeps the settings half and asserts an existing `tasks.json` is overwritten with the current chain; `TestWriteVSCodeConfigRegistersInGitignore` becomes a test that the interactive chain creates no `.gitignore` and leaves an existing one byte-identical.
- Delete `internal/ideengine/spawn_test.go`: its `TestSpawn` runs `Spawn` on bare temp directories without git, which the tracked check now refuses; its chain-stamping assertion moves into the new integration file.
- New `spawn_clean_integration_test.go` (`//go:build integration`, serial because it swaps `CodeLauncher`, following `spawn_driven_hub_integration_test.go`'s fixtures):
  - `Spawn` on a task pair whose repo tracks a `.gitignore` leaves `git status --porcelain` empty, with `.vscode/` ignored through `info/exclude`, or no exclude line written when the repo already ignores `.vscode/`.
  - The generated `tasks.json` stamps the reed chain with absolute paths and `--unless-name orch`.
  - `Spawn` overwrites an untracked `tasks.json` holding an old chain and leaves an existing `settings.json` byte-identical.
  - `Spawn` with a tracked `tasks.json` leaves it untouched, writes no `settings.json` even when absent, writes no `info/exclude` line, and still launches.

Docs in this commit:

- `docs/overview.md`, the **ide** bullet: the folder-open chain's add row carries `--unless-name orch`; `Spawn` regenerates `tasks.json` on every spawn and keeps `.vscode/` out of git through `info/exclude`, leaving a tracked `tasks.json` alone with a warning; drop "An existing worktree keeps its current tasks.json because WriteConfig never clobbers", the manual-upgrade sentence and "Plain lyx ide spawn is unchanged".
- `CONSTRAINTS.md`, Fabric Git Invariant (D6): extend the junction-exclusion bullet from junctions to every warp-side writer: lyx's own code never writes a `.gitignore` in a warp worktree, the prime included, and every ignore it needs there goes to `.git/info/exclude` through `fabricengine`; the one weft-side `.gitignore` writer is `boardengine`'s lock and manifest patterns in `_board`, the weft's `main` worktree.

**ImpactSummary:** `lyx ide spawn` changes what it writes under `.vscode/` and stops touching `.gitignore`; `SpawnDriven` keeps its behavior through the shared helper; `internal/gitignore` loses its only caller.
