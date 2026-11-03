# Codex trial: pane, sends, creel by the pane path, and load

`codex-trial-pane-load` · Task · research, codex, shuttle, creel · draft

Depends on: [`codex-trial-hooks-guards-skills`](design-codex-trial-hooks-guards-skills.md)

## Goal

Test parts 5 and 7 of project `codex-engine-trial` against real Codex sessions launched through `lym shuttle`: the pane, sends, creel by the pane path, and load, as `internal/shuttleengine/codexengine/doc.go` and `internal/creelengine/doc.go` describe them.

## Cases

Sessions run on `luna[medium]`, in this task's worktree.

1. Pane readings: the composer, the busy line and the folder-trust prompt read right through `lym shuttle state`, once each.
2. Sends: a 4,000-character send and a 40-line send through the guarded paste-then-Enter land whole; five sends each.
3. Exit: a Codex exit reads as a dead pane.
4. Creel by the pane path, in one opened session:
   - a program's notice is typed in full while the session is idle;
   - a post from an agent strand arrives as a `run lym creel read` reminder;
   - the session runs `lym creel read` and `lym creel post --reply <id>` from inside its `workspace-write` sandbox and reaches the creel daemon.
5. Load: three Codex sessions at once, each through ten turns, with one interrupt and one send during a busy turn in each; every turn end, interrupt and send reads right.
6. End to end: one darn run of a small change in a throwaway sandbox hub, with `code_assist_reviewer_model` set to `luna[medium]` for that run's `run_settings` only, followed to Code-Review's end.

## Deliver

- A report at `docs/research/codex-engine-trial-pane-load-2026-11-03.md`: per case, what was run, what was seen and whether it matches the docs.
- Each lym fault as a bug note on the board (`kind: note`, label `bug` and the module labels), one finding per note.
- No config of this hub changes; case 6 sets its model in its own run's settings only.
