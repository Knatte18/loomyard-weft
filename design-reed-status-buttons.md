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

## Placement: options discussed on 2026-10-15, none chosen

Reed's status bar today has two lines: line 0 holds the window buttons (overview, agents, shell), line 1 the session buttons and the time.
The action buttons must not mix with the window buttons.
The operator worries that a separate line next to the others invites misclicks, above all on actions that cannot be undone, such as `lyx orch distill` and `lyx orch refresh`.

1. One menu button (my recommendation, the orch's call, not the operator's): a single button with a distinct look opens a tmux `display-menu` holding the configured actions.
   A misclick only opens the menu.
   The follow toggle, which is harmless to click, can sit beside it as a button of its own.
2. A third status line of action buttons, with `confirm-before` on the destructive ones.
   It is quicker to use, but each misclick costs a dismissal.
3. The right side of line 1, before the time: no extra row, but cramped with many sessions and buttons.

## Operator view, not a decision (2026-10-15)

Prototype the placement and the menu by hand in a separate throwaway tmux server, never in a live reed session, before this becomes a task.
