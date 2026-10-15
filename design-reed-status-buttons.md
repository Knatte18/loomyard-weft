# Configurable action buttons on the reed status bar

`reed-status-buttons` · Note · feature, reed, config

## Idea

tmux status lines support clickable ranges (`#[range=user|<name>]` with a `MouseDown1Status` binding), and reed already uses them for its window buttons.
The same mechanism can carry action buttons that run a command when clicked.

Examples the operator named on 2026-10-15:

- open the pair in VS Code (`lyx ide spawn <slug>`),
- `lyx orch distill` and `lyx orch refresh` for the orch session,
- the follow toggle of note `viewer-off-shell`,
- "and much else".

## Operator view, not a decision (2026-10-15)

The button set must be highly and easily configurable, not a fixed list in code.

## Open design points

- Where the set lives: presumably reed's module config, with a hub default and per-session overrides (the orch session needs other buttons than a task pair).
- What a button entry holds: a label, the command, and which sessions show it.
- How a command runs: `run-shell` in the pair's worktree, with its output and failure visible somewhere.
- Whether the follow toggle is a built-in entry in the same set.
