# Codex trial: hooks, shell guards and skills

`codex-trial-hooks-guards-skills` · Task · research, codex, shuttle · draft

Depends on: [`codex-trial-home-sandbox`](design-codex-trial-home-sandbox.md)

## Goal

Test parts 3, 4 and 6 of project `codex-engine-trial` against real Codex sessions launched through `lym shuttle`: hooks, shell guards and skills, as `internal/shuttleengine/codexengine/doc.go` describes them.

## Cases

Each case runs once, on `luna[medium]`, in this task's worktree, unless it names two repeats.

1. Hooks: one `lym shuttle run` turn and one `lym shuttle open` session; SessionStart, UserPromptSubmit, Stop with the reply, and PostToolUse each reach the run's events file or activity stamp, with the payloads named.
2. PermissionRequest: a turn that asks for an approval reaches the events file, or the report shows why `-a never` never raises one.
3. Interrupt: a turn interrupted mid-tool-call reads as interrupted through the rollout's newest `turn_aborted`; two repeats.
4. Context size: the reading from the rollout's newest `token_count` matches the rollout after three turns.
5. Shell guards: every installed row of the shell guard table, one command each that it must deny; each is denied through the PreToolUse hook with its Codex steer, and the prompt file opens with the rows' notices.
6. A guard hook that fails to run, by a hook command made to exit non-zero in a throwaway copy, is seen failing open, and the report says how that shows.
7. Skills: a load of one `<plugin>:<name>` skill under `skill_roots` reads the newest version's `SKILL.md` and is seen loaded in the rollout; a load of a skill under no root is reported unknown.

## Deliver

- A report at `docs/research/codex-engine-trial-hooks-guards-skills-2026-11-03.md`: per case, what was run, what was seen and whether it matches the package doc.
- Each lym fault as a bug note on the board (`kind: note`, label `bug` and the module labels), one finding per note.
