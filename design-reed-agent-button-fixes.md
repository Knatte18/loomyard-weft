# Reed agent button: explain the name rule and start the agent with lym context

`reed-agent-button-fixes` · Task · bug, reed, stencil · running · Code-Burler-Review

## Seen (operator, 2026-11-27)

The operator tried the agent button in reed's status bar for the first time.

1. It refused a name with spaces without saying what a valid name is.
2. The agent it spawned knew nothing of lym: no skills, no commands, no idea what it is, which hub, worktree or task it serves, or what it is for.

## Task

- Name the rule for the agent's name when asking for it; on a refusal name the rule and ask again, or turn spaces into dashes.
- Spawn the agent with an opening stencil that says what it is, which hub, worktree and task (or the prime) it sits in and what it is for,
  and which skills to load for that context, such as `ly:cli`, `ly:board` and the scribe skills.
  Give it the same instructions, guards and skills any other lym session of its engine gets.
