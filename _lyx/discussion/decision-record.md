# batten opens VS Code on the new pair with reed attached

## Goal

When batten starts a task, a VS Code window opens on the new task worktree with the child run's reed session already attached in its terminal,
so the operator watches the run without typing any command.

## Scope

In:

- Batten's `Run-Shed` row opens VS Code on the task worktree once per run, right after its first successful spawn of the child's bootstrap (`lyx loom start --no-attach`), which is the point the child's driver strand is up.
- A driven-pair variant of the ide spawn path in `internal/ideengine`, reached by batten's CLI wiring (`internal/battencli`) through a new injected closure on `battenshed.InnerRunDeps`.
- A second generated `tasks.json` shape in `internal/vscode`: a `folderOpen` chain that runs `lyx reed attach` only.
- Launch failure degrades to a warning; the lifecycle continues.
- Docs: `docs/overview.md`'s batten entry and ide entry, and any `manifest/designs/` doc covering batten or ide, in the same commit.

Out:

- Plain `lyx ide spawn <slug>` keeps its current behaviour: the `reed up` → `reed add claude` → `reed attach` chain, write-only-when-absent config, and its `.gitignore` edit.
- No detection of "driven" pairs inside `ide spawn`; the caller decides which chain it wants.
- No recipe row, config key or CLI flag to switch the IDE open off.
- No new batten recipe row; `contracts/recipes/batten-recipe.yaml` row names and order are unchanged.

## Decisions

- **The strand-is-up point is `Run-Shed`'s spawn, not `Seed-Child`.**
  Rationale: `Seed-Child` only writes, commits and pushes the child's seed;
  `Run-Shed`'s `Spawn` closure runs `lyx loom start --no-attach` in the task worktree and blocks until that bootstrap returns, which happens once the child's driver is up.
- **The open is a new injected closure on `battenshed.InnerRunDeps` (e.g. `OpenIDE func(ctx) error`), called by the `InnerRun` producer after a spawn that returned success, not a new recipe row.**
  Rationale: the only point after the strand is up and before the child finishes is inside `Run-Shed`;
  a separate row would require splitting `Run-Shed`'s spawn out of it, duplicating its spawn-confirmation and re-spawn logic across two rows and changing the durable row set resume depends on.
  The step stays a batten lifecycle step, not part of the loom run.
- **Opened once per run, gated by a marker under `Run-Shed`'s own scratch directory, beside the existing `-spawn-confirmed`, `-approval-acted` and `-done-seen` markers.**
  The marker is written after the open is attempted, whatever its outcome.
  Rationale: `Run-Shed` also re-spawns the child on an approved `awaiting` state and on a running child with no confirmed spawn;
  re-opening then would resurrect a window the operator closed deliberately, and retrying a failed launch would warn on every re-spawn of a headless run.
- **Open failure is a warning, never a verdict change.**
  `InnerRun` logs the closure's error through `logger.Warn` and continues to the status read exactly as if the open succeeded.
  Rationale: the brief requires the lifecycle to continue with no `code` on `PATH` or on a headless machine.
- **`internal/ideengine` gains a driven-pair spawn entry point; `vscode.WriteConfig` gains a chain selector with two values: today's interactive chain and an attach-only chain.**
  The attach-only `tasks.json` holds one `reed attach` task (`lyx reed attach`, the absolute `lyx` path with the same bare-name fallback) marked `runOn: folderOpen`, and no `reed up` or `reed add claude` task.
  Rationale: `reed attach` already boots a cold session itself (`EnsureSession`), and `loom start` has brought reed up by this point, so `reed up` is redundant;
  `reed add claude` adds a second, bare claude strand beside the driver, which is the bug being fixed.
- **The driven path always writes `tasks.json`, overwriting an existing one; `settings.json` keeps the write-only-when-absent rule.**
  Rationale: `.vscode/` in a pair batten just created is lyx-generated, and a leftover interactive chain (from an `ide spawn` run against the pair before batten reached `Run-Shed`) would reproduce the bare-claude-strand bug;
  settings carry only color and title and are harmless to keep.
- **The driven path never modifies a tracked file.**
  It does not call `gitignore.Ensure` on the task worktree's `.gitignore`;
  `.vscode/` is kept out of git through the task worktree's `.git/info/exclude` instead (or left alone when git already ignores it).
  Rationale: the child's loom run commits in that worktree, so a lyx-managed block appended to `.gitignore` would either be swept into the task's commits or trip a clean-tree check.
- **The chain choice does not depend on the child's driver.**
  Both `go` and `llm` children get the attach-only chain;
  nothing reads the seed's `driver` field.
  Rationale: either driver's session already carries its own driving surface, and the Driver Choice Single-Site Invariant forbids a second reader of the recorded driver.
- **Batten opens the bare task folder `<hub>/<slug>/<AnchorRel>`, with the title-bar color picked the same way `ide spawn` picks it.**
  Rationale: a batten slug is never the prime, so the prime-workspace branch of `ideengine.Spawn` never applies.

## Constraints

- **Told-Geometry Invariant:** `internal/battenshed` stays told — it imports neither `internal/ideengine` nor `internal/lyxcwd`;
  the new closure is resolved and built in `internal/battencli/wire.go`, which holds the prime's `lyxcwd.Location`.
  `battenshed`'s `seam_enforcement_test.go` must still pass.
- **Driver Choice Single-Site Invariant:** no new reader of the seed's `driver` field; `internal/loomcli/bootstrap_test.go`'s tripwire needs no new carve-out.
- **Batten recipe row names are durable resume identities** (`internal/battenrecipe` coverage guard); they do not change.
- **Durable-vs-Ephemeral State Invariant:** the once-marker lives in `Run-Shed`'s existing scratch directory (ephemeral), never under `_lyx`.
- **CLI/Cobra Invariant:** no new command or flag; `lyx ide spawn`'s help text stays accurate.
- **Documentation Lifecycle:** module docs and `docs/overview.md` land in the same commit.
  No new cross-cutting invariant, so `CONSTRAINTS.md` is untouched unless the plan finds one.
- Tests never launch a real VS Code: batten tests stub the new `InnerRunDeps` closure, and any test crossing `ideengine` stubs `ideengine.CodeLauncher`.

## Auto-mode assumptions

- No operator answered questions; every open point in the brief was self-picked (see the support log's question ledger).
- Assumed `code <folder>` is an acceptable launch on every platform batten runs on; on Linux a headless `code` that fails after a successful process start is not detected, and that counts as degraded, not failed.
- Assumed no operator needs to switch the open off per run; the headless degrade covers CI and remote machines.
- Assumed nobody hand-edits `.vscode/tasks.json` in a pair batten created, so overwriting it on the driven path loses nothing.

## Open risks

- A Linux machine with `code` on `PATH` but no display starts the process successfully, so no warning is logged and nothing opens; the operator sees no signal.
- `.git/info/exclude` is shared by every worktree of the repo, so the driven path's exclude entry applies repo-wide; it is idempotent and only hides `.vscode/`, which lyx already treats as untracked config.
- A run resumed from an older build past its first spawn has no once-marker and no further first spawn, so no window opens for it.
- If the child's driver dies before VS Code finishes starting, the attach boots an empty session; the operator sees an empty pane rather than an error.

## Acceptance criteria

- A batten run's `Run-Shed`, on its first successful child spawn, invokes the open closure exactly once, after the spawn returns and before the post-spawn status read decides the row's outcome.
- A second spawn in the same run (approved `awaiting` resume, or running-without-confirmation re-spawn) does not invoke the open closure again.
- An open closure returning an error leaves `Run-Shed`'s outcome, reason and bounce accounting identical to the success case, and emits one warning log line naming the slug and the error.
- A failed spawn does not invoke the open closure.
- The driven-pair `tasks.json` contains exactly one task, running `lyx reed attach` on `folderOpen`; it contains no `reed up` and no `reed add` task. Covered by a `internal/vscode` test.
- The driven path overwrites an existing interactive `tasks.json` and keeps an existing `settings.json` byte-for-byte. Covered by a test.
- The driven path leaves the task worktree's tracked tree clean: `git status --porcelain` is empty after it runs in a fresh pair (hub integration test via `hubforge`), and `.vscode/` is ignored.
- Plain `lyx ide spawn <slug>` output is unchanged: its existing tests pass unmodified.
- `ideengine.CodeLauncher` is stubbed in every test that reaches the driven spawn; no test starts a real `code` process.
- `battenshed`'s seam enforcement test, `battenrecipe`'s coverage guard and `loomcli`'s driver-field tripwire pass unchanged.
- `docs/overview.md`'s batten and ide entries describe the open step, its once-per-run gate and its warning-only failure mode.

## Notes for the plan writer

- `Run-Shed`'s spawn site and its markers live in `internal/battenshed/innerrun.go` (`Call`, `SpawnConfirmedFile`, the `-approval-acted` and `-done-seen` helpers); the approval re-spawn is in `callAwaiting`.
- The spawn closure and the other `InnerRunDeps` closures are wired in `internal/battencli/wire.go`; `DriverAlive` there shows how a closure resolves the task worktree on Call.
- `internal/ideengine/spawn.go`'s `Spawn` resolves the worktree path, color and binary paths; the driven entry can share all of that except the chain and the `.gitignore` step.
- `internal/vscode/config.go`'s `WriteConfig` is the only `tasks.json` writer.
- `fabricengine` already writes `.git/info/exclude` entries for `.lyx`; check whether a helper there fits before adding one.
