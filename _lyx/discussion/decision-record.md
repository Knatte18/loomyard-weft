# Decision record — seed-ide-interactive-chain

## Goal

Opening any lyx-seeded worktree in VS Code starts nothing: no `reed up`, no `reed add` of a `claude` strand and no automatic `reed attach`.
Batten's Seed-IDE row on a driven pair and `lyx ide spawn` on the prime or a task pair write the same `.vscode/` config through one code path.

## Scope

- In: `internal/vscode`'s `WriteConfig` and its tasks.json generation; `internal/ideengine`'s `Spawn`, `SeedTask` and their shared config writer; their tests; the docs that describe the folderOpen chain (`docs/overview.md`'s ide row, `internal/loomcli/doc.go`'s operator-session paragraph, the `internal/vscode` and `internal/ideengine` file and symbol docs).
- Out: reed's `add` verb and its `--unless-name` / `--if-absent` flags, which stay as operator-facing reed features; the keybindings.json seeding; the hub workspace file; batten's recipe rows and flags.

## Decisions

- **Decision:** the generated tasks.json carries no `runOptions.runOn: folderOpen` task at all.
  **Rationale:** the operator's decision: runs are watched in their own terminal window, and `reed add claude` must never run from inside VS Code.
- **Decision:** tasks.json holds one on-demand task, labelled `reed attach`, running the absolute `lyx` path with `reed attach`, presented in a new focused terminal.
  **Rationale:** the operator welcomes a VS Code shortcut for `lyx reed attach` run on demand; a plain task is reachable from "Tasks: Run Task" and starts nothing on folder open.
- **Decision:** `vscode.TaskChain`, `TaskChainInteractive`, `TaskChainAttachOnly` and the `claudePath` parameter are deleted, and `WriteConfig` writes the one tasks.json shape.
  **Rationale:** one VS Code seeding shared by every caller, with no unused variant left behind; nothing in the config resolves `claude` any more.
- **Decision:** `Spawn` and `SeedTask` keep sharing a single config writer in `internal/ideengine`, renamed off "interactive", which stamps only the absolute `lyx` path.
  **Rationale:** the operator asked for no duplicated seeding code, and the old name describes a chain that no longer exists.
- **Decision:** tasks.json stays regenerated on every write, overwriting an untracked file, and a tracked tasks.json is still left alone with a warning.
  **Rationale:** the overwrite is what replaces the old folderOpen chain in existing worktrees on their next seed or spawn; the tracked-file guard is unchanged behavior.

## Constraints

- `PATTERN-told-geometry`, `PATTERN-cwd-resolution`: no new path derivation; the existing `lyxcwd.Location` plumbing is kept.
- `PATTERN-test-economy`: change existing tests to pin the new shape rather than adding parallel ones.
- `PATTERN-comment-line-breaks`: Go comments one sentence per line.
- `PATTERN-documentation-lifecycle` and the repo's docs rule: `docs/overview.md`'s ide row changes in the same commit, since observable CLI behavior changes.

## Auto-mode assumptions

- The on-demand `reed attach` task is the shortcut the operator's view asks for; no keybindings.json entry is added for it.
- `reed attach` boots a cold worktree's session itself, so the on-demand task needs no preceding `reed up` task.
- Removing `--unless-name` and `--if-absent` from reed is out of scope: they remain usable reed flags with no lyx-generated caller.

## Open risks

- A worktree already holding the old chain keeps it until its next `lyx ide spawn` or Seed-IDE; opening it in VS Code before that still runs the old chain.

## Acceptance criteria

- `vscode.WriteConfig` writes a tasks.json with exactly one task, labelled `reed attach`, with the stamped absolute `lyx` command, args `reed attach`, and no `runOptions` and no `dependsOn`.
- An empty `lyx` path still falls back to the bare name `lyx`.
- tasks.json is overwritten on every call; settings.json is written only when absent.
- `ideengine.SeedTask` and `ideengine.Spawn` both produce that tasks.json, with no task naming `claude` or `reed add`, through the one shared writer.
- The tracked-tasks.json skip and the info/exclude handling behave as before.
- `docs/overview.md` and `internal/loomcli/doc.go` no longer describe a folderOpen chain.
- `go build` passes with `TaskChain` and its constants deleted.

## Notes for the plan writer

- Chain generation: `internal/vscode/config.go` (`WriteConfig`, `writeInteractiveTasks`, `writeAttachOnlyTasks`).
- Shared writer: `internal/ideengine/spawn.go` (`writeInteractiveConfig`, `writeVSCodeConfig`).
- Tests pinning the old chain: `internal/vscode/config_test.go`, `internal/ideengine/spawn_clean_integration_test.go`.
