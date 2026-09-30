---
format: 5
approved: false
language: go
---

# Plan: batten opens VS Code on the new pair with reed attached

When batten's `Run-Shed` row first spawns a task worktree's child run successfully, it opens VS Code on that worktree's anchor folder with a `folderOpen` task that runs `lyx reed attach` into the child's own session, so the operator watches the run without typing a command.
The open is a new injected closure on `battenshed.InnerRunDeps`, gated once per run by a marker in `Run-Shed`'s scratch directory, and a failed launch is a warning only.
The closure calls a new driven-pair entry point in `internal/ideengine` that writes an attach-only `tasks.json` and keeps `.vscode/` out of git through the shared `info/exclude`, never touching a tracked file.
Plain `lyx ide spawn` is unchanged.

## Card Index

1 — vscode-attach-chain — add the attach-only chain selector to `vscode.WriteConfig`
2 — fabric-anchored-exclude — export an anchored, lock-held `info/exclude` entry writer and a tracked-path query
3 — ide-spawn-driven — add `ideengine.SpawnDriven`, the driven-pair open path
4 — innerrun-open-once — call a new `OpenIDE` closure once per run after a successful spawn
5 — batten-wire-open — wire `OpenIDE` to `SpawnDriven` in battencli and document the step

## Shared Decisions

### Decision: no-real-editor-in-tests

- **Decision:** No test starts a real `code` process: `battenshed` tests stub `OpenIDE`, every test reaching `SpawnDriven` stubs `ideengine.CodeLauncher`, and `battencli`'s integration `TestMain` replaces it for that whole binary.
- **Rationale:** the lifecycle integration tests keep the real `OpenIDE` wiring, so a per-test stub would be easy to miss.
- **Applies to:** cards 3, 4, 5

### Decision: git-stays-in-fabricengine

- **Decision:** The driven path's git calls (the exclude write and the tracked-file query) live in `fabricengine`, and every git-touching test of them is integration-tagged.
- **Rationale:** the Fabric Git Invariant routes git through `fabricengine`, the exclude write must hold its existing lock, and the Test Tier Purity Invariant bars git spawns in untagged tests.
- **Applies to:** cards 2, 3

## verify:

go build ./...
go vet ./internal/vscode/... ./internal/fabricengine/... ./internal/ideengine/... ./internal/battenshed/... ./internal/battencli/...
go test ./internal/vscode/... ./internal/fabricengine/... ./internal/ideengine/... ./internal/battenshed/... ./internal/battencli/... ./internal/battenrecipe/... ./internal/loomcli/...
go test -tags integration ./internal/vscode/... ./internal/fabricengine/... ./internal/ideengine/... ./internal/battenshed/... ./internal/battencli/...
