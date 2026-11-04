# P2 Stress test of codexengine: every designed capability, hooks above all

Rendered 2026-11-04 11:22 UTC; every age below is as of then.
Under: [Board](README.md).

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 16h · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

## Goal (operator, 2026-11-03)

`codexengine` landed with project `codex-engine` and has run motif runs well, but was never tested as a whole.
A stress test proves, on real launches, that the engine does everything its design says, the hooks above all, and records each fault as a bug note.

## What the engine is designed to do

The package doc `internal/shuttleengine/codexengine/doc.go` is the spec; each part below tests one area of it against real Codex sessions launched through `lym shuttle`.

1. Home and trust: the shared lym Codex home under `CODEX_HOME`, `auth.json` and sessions linked to the operator's home, `config.toml` rendered whole under a file lock, trusted roots for a worktree and a linked worktree, dropped roots for a gone folder, the read-back Warn, and the folder-trust prompt backstop.
2. Sandbox and refusals: `-a never -s workspace-write`, the records checkout as a writable root, a committed `.codex/rules` refusing the launch and a committed `.codex/config.toml` named in a Warn.
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

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Codex trial: home, trust, sandbox and refusals**<br>• Seven cases on real Codex launches against the package doc | [`codex-trial-home-sandbox`](design-codex-trial-home-sandbox.md) | darn · docs |  | 15h<br>15h | codex, shuttle |
| T5 | `draft` **Codex trial: hooks, shell guards and skills**<br>• Seven cases on real Codex launches against the package doc | [`codex-trial-hooks-guards-skills`](design-codex-trial-hooks-guards-skills.md) | darn · docs | T3 | 15h<br>15h | codex, shuttle |
| T6 | `draft` **Codex trial: pane, sends, creel by the pane path, and load**<br>• Six cases, three sessions at once and one darn run with a Codex reviewer seat | [`codex-trial-pane-load`](design-codex-trial-pane-load.md) | darn · docs | T5 | 15h<br>15h | codex, shuttle, creel |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
