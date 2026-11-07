# P2 Codexengine: every task, trial and note about the Codex engine

Rendered 2026-11-07 15:54 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 3d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

## Goal (operator, 2026-11-03)

`codexengine` landed with project `codex-engine` and has run motif runs well, but was never tested as a whole.
A stress test proves, on real launches, that the engine does everything its design says, the hooks above all, and records each fault as a bug note.

## What the engine is designed to do

The package doc `internal/shuttleengine/codexengine/doc.go` is the spec; each part below tests one area of it against real Codex sessions launched through `lym shuttle`.

1. Home and trust: the shared lym Codex home under `CODEX_HOME`, `auth.json` and sessions linked to the operator's home, `config.toml` rendered whole under a file lock, trusted roots for a worktree and a linked worktree, dropped roots for a gone folder, the read-back Warn, and the folder-trust prompt backstop.
2. Sandbox and refusals: `-a never -s workspace-write`, the records checkout as a writable root, a `.codex/rules` folder on disk, tracked by git or not, refusing the launch and a `.codex/config.toml` on disk named in a Warn (the checks read the folders on disk since `codex-trial-1-fixes`).
3. Hooks: SessionStart, UserPromptSubmit, Stop with its reply, PermissionRequest and PostToolUse each reach the run's events file or activity stamp; an interrupt reads as interrupted through `turn_aborted`; the context size from `token_count`.
4. Shell guards: every installed guard row denies its command through the PreToolUse hook with its Codex steer, the prompt file carries the notices, and a hook that fails to run is seen failing open.
5. Pane and send: the composer, the busy line and the trust prompt read right; a long or multi-line send through the guarded paste-then-Enter lands whole; a dead pane after exit.
   Creel by the pane path, the only one a Codex strand has: a program's notice typed in full into an idle session, an agent's mail reminded with `run lym creel read`, and the Codex session running `lym creel read` and `lym creel post --reply` from inside its `workspace-write` sandbox, which must reach the creel daemon.
6. Skills: a `<plugin>:<name>` load reads the newest version's `SKILL.md` and the load is seen in the rollout; an unknown skill is reported unknown.
7. Stress: several Codex sessions at once, long sessions, repeated interrupts and sends, and a darn run with a Codex seat end to end.

## Rules

- Each part writes its findings to a report under `docs/research/`, and files each lym fault as a bug note on this board.
- Parts run as darn with profile `docs`, unless the operator says otherwise.
- A Codex run gets `high` effort at most.

## Built since (tidy, 2026-11-05)

Task `codex-creel-hooks` (`e3aedbc5`) gave Codex strands creel delivery through hooks, as Claude strands have: mail reaches a running session at the end of a tool call through `PostToolUse`, at a turn end through `Stop`, and an idle session through `codex queue` run by the strand's `hook-wait` waiter. So part 5's "the only one a Codex strand has" holds only for a strand opened without a prompt file (`OpenLaunch`, a standalone run), which stays a pane-mail strand.
Task `codex-creel-trial` (`afa2cf2e`, `docs/research/codex/engine-trial/codex-creel-trial-2026-11-05.md`) trialed it on real sessions in `lyt` and found one fault, filed as note `codex-idle-wake-extra-reminder`.

## Built since (tidy, 2026-11-07)

Task `codex-skill-load-false-missing` (`0e02aaad`) fixed a false `skill skipped ... cause="not loaded"` in part 6's load check.
`ClassifySkillLoad` now also reads a skill as loaded when a tool call output of the load turn holds the whole of its `SKILL.md`, so a seat that reads its skills through a path built at run time (a `cat .../skills/${name}/SKILL.md` loop) is no longer reported missing.
A partial output still leaves a skill missing.
The fix is checked against unit cases only: the original rollout and a live Codex run were not used, so part 6 should still see the load on a real session.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 1d<br>1d | codex, shuttle |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 4d<br>26m | shuttle, codex, loom |
