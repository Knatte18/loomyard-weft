# batten opens VS Code on the new pair with reed attached

## Goal

When batten starts a task, a VS Code window opens on the new task worktree with the child run's reed session already attached in its terminal,
so the operator watches the run without typing any command.

## Scope

In:

- Batten's `Run-Shed` row opens VS Code on the task worktree once per run, right after a successful spawn of the child's bootstrap (`lyx loom start --no-attach`), which is the point the child's driver strand is up.
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
- **The open is a new injected closure on `battenshed.InnerRunDeps`, called by the `InnerRun` producer after a spawn that returned success, not a new recipe row.**
  Rationale: the only point after the strand is up and before the child finishes is inside `Run-Shed`.
  The step stays a batten lifecycle step, not part of the loom run.
- **Opened once per run, gated by a once-marker under `Run-Shed`'s own scratch directory, beside the existing `SpawnConfirmedFile`, approval-acted and done-seen markers.**
  The gate is "no once-marker present", checked after every successful spawn at both spawn sites: `Call`'s read-before-spawn branch and `callAwaiting`'s approval re-spawn.
  The marker is written after the open is attempted, whatever its outcome.
  When `Call` finds no child status file (a fresh child), it removes any once-marker before that spawn, so a marker left by an earlier run of the same slug never suppresses a later run's open;
  scratch outlives a run, which is why the done-seen marker is already cleared the same way.
  A lost marker (the run resumed on another machine, or `.lyx` cleared) re-opens the window on the next successful spawn;
  that is intended, since the operator on the new machine has no window yet.
  Rationale: `Run-Shed` also re-spawns the child on an approved `awaiting` state and on a running child with no confirmed spawn;
  re-opening then would resurrect a window the operator closed deliberately, and retrying a failed launch would warn on every re-spawn of a headless run.
- **Open failure is a warning, never a verdict change.**
  `InnerRun` logs the closure's error through `logger.Warn` and continues to the status read exactly as if the open succeeded.
  Rationale: the brief requires the lifecycle to continue with no `code` on `PATH` or on a headless machine.
- **`internal/ideengine` gains a driven-pair spawn entry point; `vscode.WriteConfig` gains a chain selector with two values: today's interactive chain and an attach-only chain.**
  The attach-only `tasks.json` holds one `reed attach` task (`lyx reed attach`, the absolute `lyx` path with the same bare-name fallback) marked `runOn: folderOpen`, and no `reed up` or `reed add claude` task.
  Rationale: `reed attach` already boots a cold session itself (`EnsureSession`), and `loom start` has brought reed up by this point, so `reed up` is redundant;
  `reed add claude` adds a second, bare claude strand beside the driver, which is the bug being fixed.
- **The driven path writes `tasks.json` whenever it is untracked, overwriting an existing one; `settings.json` keeps the write-only-when-absent rule.**
  When the task worktree's repo tracks `.vscode/tasks.json`, the driven path leaves it untouched, logs a warning naming the slug, and still opens VS Code.
  Rationale: an untracked `.vscode/` in a pair batten just created is lyx-generated, and a leftover interactive chain (from an `ide spawn` run against the pair before batten reached `Run-Shed`) would reproduce the bare-claude-strand bug;
  a tracked one belongs to the repo, and overwriting it would dirty the tree the child commits in.
  Settings carry only color and title and are harmless to keep.
- **The driven path never modifies a tracked file.**
  It does not call `gitignore.Ensure` on the task worktree's `.gitignore`;
  `.vscode/` is kept out of git through the repository's shared `info/exclude` instead (or left alone when git already ignores it).
  Rationale: the child's loom run commits in that worktree, so a lyx-managed block appended to `.gitignore` would either be swept into the task's commits or trip a clean-tree check.
- **The chain choice does not depend on the child's driver.**
  Both `go` and `llm` children get the attach-only chain;
  nothing reads the seed's `driver` field.
  Rationale: either driver's session already carries its own driving surface, and the Driver Choice Single-Site Invariant forbids a second reader of the recorded driver.
- **Batten opens the bare task folder `<hub>/<slug>/<AnchorRel>`, with the title-bar color picked the same way `ide spawn` picks it.**
  That folder must be the directory `Run-Shed`'s spawn runs the child's bootstrap in (the task worktree's anchor path), so the `folderOpen` `reed attach` resolves the child's own reed session rather than a fresh one.
  Rationale: a batten slug is never the prime, so the prime-workspace branch of `ideengine.Spawn` never applies;
  reed resolves its session from the working directory's location.

## Constraints

- **Told-Geometry Invariant:** `internal/battenshed` stays told — it imports neither `internal/ideengine` nor `internal/lyxcwd`;
  the new closure is resolved and built in `internal/battencli/wire.go`, which holds the prime's `lyxcwd.Location`.
  `battenshed`'s `seam_enforcement_test.go` must still pass.
- **Driver Choice Single-Site Invariant:** no new reader of the seed's `driver` field; `internal/loomcli/bootstrap_test.go`'s tripwire needs no new carve-out.
- **Batten recipe row names are durable resume identities** (`internal/battenrecipe` coverage guard); they do not change.
- **Durable-vs-Ephemeral State Invariant:** the once-marker lives in `Run-Shed`'s existing scratch directory (ephemeral), never under `_lyx`.
- **Shared exclude file:** git resolves `info/exclude` to the repository's common gitdir, so one file serves every worktree of the repo, and `lyx fabric` verbs and concurrent batten runs edit it too.
  The driven path resolves it with `git rev-parse --git-path info/exclude` and writes it only under `fabricengine`'s existing exclude lock with an atomic replace, by exporting or wrapping that code rather than adding a second locking scheme.
  Its entry must match `.vscode/` at the anchor subpath, not only at the repo root.
- **CLI/Cobra Invariant:** no new command or flag; `lyx ide spawn`'s help text stays accurate.
- **Documentation Lifecycle:** module docs and `docs/overview.md` land in the same commit.
  No new cross-cutting invariant, so `CONSTRAINTS.md` is untouched unless the plan finds one.
- Tests never launch a real VS Code: batten tests stub the new `InnerRunDeps` closure, and any test crossing `ideengine` stubs `ideengine.CodeLauncher`.

## Auto-mode assumptions

- No operator answered questions; every open point in the brief was self-picked (see the support log's question ledger).
- Assumed `code <folder>` is an acceptable launch on every platform batten runs on; on Linux a headless `code` that fails after a successful process start is not detected, and that counts as degraded, not failed.
- Assumed no operator needs to switch the open off per run; the headless degrade covers CI and remote machines.
- Assumed nobody hand-edits an untracked `.vscode/tasks.json` in a pair batten created, so overwriting it on the driven path loses nothing.

## Open risks

- A Linux machine with `code` on `PATH` but no display starts the process successfully, so no warning is logged and nothing opens; the operator sees no signal.
- The shared `info/exclude` entry applies repo-wide: it hides an untracked `.vscode/` in every worktree of the repo, not only the task worktree.
- A run resumed from an older build has no once-marker, so it opens a window on its next successful spawn; a run already past its last spawn opens none.
- If the child's driver dies before VS Code finishes starting, the attach boots an empty session; the operator sees an empty pane rather than an error.

## Acceptance criteria

- A batten run's `Run-Shed`, on its first successful child spawn, invokes the open closure exactly once, after the spawn returns and before the post-spawn status read decides the row's outcome.
- A second spawn in the same run (approved `awaiting` resume, or running-without-confirmation re-spawn) does not invoke the open closure again.
- With no once-marker present, a successful approval re-spawn in `callAwaiting` invokes the open closure once.
- A once-marker left in scratch by an earlier run of the same slug does not suppress the open when `Call` finds no child status file.
- An open closure returning an error leaves `Run-Shed`'s outcome, reason and bounce accounting identical to the success case, and emits one warning log line naming the slug and the error.
- A failed spawn does not invoke the open closure.
- The driven-pair `tasks.json` contains exactly one task, running `lyx reed attach` on `folderOpen`; it contains no `reed up` and no `reed add` task. Covered by a `internal/vscode` test.
- The driven path overwrites an existing untracked interactive `tasks.json` and keeps an existing `settings.json` byte-for-byte. Covered by a test.
- The driven path leaves a tracked `.vscode/tasks.json` byte-for-byte unchanged and logs one warning naming the slug. Covered by a test.
- The driven path leaves the task worktree's tracked tree clean: `git status --porcelain` is empty after it runs in a fresh pair, on both a root-anchored and a subpath-anchored hub (hub integration test via `hubforge`), and `.vscode/` is ignored.
- The driven path's exclude write holds `fabricengine`'s exclude lock, and preserves every existing line of the shared `info/exclude`.
- The folder passed to the stubbed `ideengine.CodeLauncher` is `<hub>/<slug>/<AnchorRel>`, the same directory the spawn closure runs the bootstrap in.
- Plain `lyx ide spawn <slug>` output is unchanged: its existing tests pass unmodified.
- `ideengine.CodeLauncher` is stubbed in every test that reaches the driven spawn; no test starts a real `code` process.
- `battenshed`'s seam enforcement test, `battenrecipe`'s coverage guard and `loomcli`'s driver-field tripwire pass unchanged.
- `docs/overview.md`'s batten and ide entries describe the open step, its once-per-run gate and its warning-only failure mode.

## Notes for the plan writer

- `Run-Shed`'s spawn site and its markers live in `internal/battenshed/innerrun.go` (`Call`, `SpawnConfirmedFile`, the `-approval-acted` and `-done-seen` helpers); the approval re-spawn is in `callAwaiting`.
- The spawn closure and the other `InnerRunDeps` closures are wired in `internal/battencli/wire.go`; `DriverAlive` there shows how a closure resolves the task worktree on Call.
- `internal/ideengine/spawn.go`'s `Spawn` resolves the worktree path, color and binary paths; the driven entry can share all of that except the chain and the `.gitignore` step.
- `internal/vscode/config.go`'s `WriteConfig` is the only `tasks.json` writer.
- `fabricengine`'s exclude lock and atomic replace live in `internal/fabricengine/gitexclude.go` (unexported today); its junction entries are root-anchored, which a `.vscode/` entry must not copy.
