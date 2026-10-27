# Configurable action buttons on the reed status bar

`reed-status-buttons` · Note · feature, reed, config, batten

## Idea

tmux status lines support clickable ranges (`#[range=user|<name>]` with a `MouseDown1Status` binding), and reed already uses them for its window buttons.
The same mechanism can carry action buttons that run a command when clicked.

Examples the operator named on 2026-10-15:

- open the pair in VS Code (`lym ide spawn <slug>`),
- `lym orch distill` and `lym orch refresh` for the orch session,
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
The operator worries that a separate line next to the others invites misclicks, above all on actions that cannot be undone, such as `lym orch distill` and `lym orch refresh`.

1. One menu button (my recommendation, the orch's call, not the operator's): a single button with a distinct look opens a tmux `display-menu` holding the configured actions.
   A misclick only opens the menu.
   The follow toggle, which is harmless to click, can sit beside it as a button of its own.
2. A third status line of action buttons, with `confirm-before` on the destructive ones.
   It is quicker to use, but each misclick costs a dismissal.
3. The right side of line 1, before the time: no extra row, but cramped with many sessions and buttons.

## Operator view, not a decision (2026-10-15)

Prototype the placement and the menu by hand in a separate throwaway tmux server, never in a live reed session, before this becomes a task.

## Prototype (folded in from note `reed-status-buttons-prototype`, 2026-10-24)

Operator decision (2026-10-15): a fresh agent builds the prototype when the operator says so, not before.
- A separate throwaway tmux server (its own `-L` socket), never reed's server and never the prime's session `loomyard`.
- A status bar shaped like reed's two lines, with one menu button opening a `display-menu` of sample actions (option 1), a follow toggle button (note `viewer-off-shell`), and a third line of action buttons, the destructive ones behind `confirm-before` (option 2).
- The sample actions only echo or display a message; none runs a real `lym` verb.
- The agent opens the server in a Konsole window for the operator to click through, and tears it down when the operator is done.
- The tmux config used goes into git under `docs/research/` through a docs-only commit, so the chosen layout can seed the task.

## A terminal-window toggle (folded in from note `open-window-toggle`, 2026-10-24)

Whether batten opens a desktop terminal window on a task worktree is decided by `batten.yaml`'s `open_window`, read from the template and the board default layer only, and `lym batten run --window` forces it on whatever that says.
The operator has no quick way to turn the windows off, although each run is a window of the prime's reed session.
Idea: a button in the orch tab's status bar beside `follow` that says whether runs open their terminal window automatically; when it is off, `Open-Window` opens nothing, `--window` included, and the reopen of a resumed run's window too.
The `follow` button keeps its state in the tmux user option `@lym_follow` (`internal/reedengine/bindings.go`), which dies with the tmux server; this switch must survive a restart, so it needs a machine-local home batten can read.
Operator wish, 2026-10-22; the design is open.
