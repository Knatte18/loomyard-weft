# Report the Codex trial of hooks, shell guards and skills

Adds `docs/research/codex-engine-trial-hooks-guards-skills-2026-11-03.md`, the report of parts 3, 4 and 6 of project `codex-engine-trial`, run on real Codex sessions (CLI 0.164.0, `luna[medium]`) launched through `lym shuttle`.
No code or config changes.

## What the report says

Each of the seven cases records what was run, what was seen and whether it matches the `codexengine` package doc.
All seven match:

- Hooks: a run's events file holds SessionStart, UserPromptSubmit and Stop with the reply, and PostToolUse touches the activity stamp; an opened session carries no hooks.
- PermissionRequest: none is raised, since `-a never` turns a sandbox refusal into a tool error.
- Interrupt: three interrupted runs fire no Stop, and each rollout's newest `turn_aborted` reads interrupted.
- Context size: after three turns the engine's reading equals the rollout's newest `token_count`.
- Shell guards: every installed row denies its command through PreToolUse with the Codex steer, and the prompt file opens with the rows' notices.
- A failing guard hook fails open and shows only as `Hook failed` lines in the pane.
- Skills: a skill under a root is read from its newest version, and one under no root is reported unknown.

## Faults found

Two lym faults outside the doc's claims are filed as board notes, label `bug`:

- B4 `shuttle-hidden-anchor-dies`: `lym shuttle run --anchor hidden` ends died on Codex and Claude alike.
- B5 `codex-state-always-unknown`: `lym shuttle state` reads every Codex run as `unknown`, which hides a running turn and an interrupt.

## Design decisions

- Cases 6 and 7 use throwaway builds, `go build -overlay` over scratch copies of one file each, because no shipped verb fails a guard hook or names skills; the worktree's tracked files stay unchanged and the builds were deleted.
- The interrupt, the later turns of case 4 and the opened session's turn were sent with `tmux send-keys`, since shuttle has no verb for them; the report names this.
- Case 3c's pane showed no background terminal at the interrupt, so that repeat may have landed before its command started; the report says so.
- Case 2 did not fire a PermissionRequest hook: the launch line fixes the approval policy, and no verb offers another.
