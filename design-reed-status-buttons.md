# Configurable action buttons on the reed status bar

`reed-status-buttons` · Note · feature, reed, config, batten

## Idea

tmux status lines support clickable ranges (`#[range=user|<name>]` with a `MouseDown1Status` binding), and reed already uses them for its window buttons.
The same mechanism can carry action buttons that run a command when clicked.

Buttons the operator named:

- HALT for a run: `lym tender pause <slug>`, which holds agent and network work and interrupts the running turns at once; `lym tender resume <slug>` lifts it.
  `lym batten pause <slug>` waits for the next producer boundary, which can take very long, so it is no halt.
  Today the operator has no quick way to do either from where they watch the run.
- Open the pair in VS Code (`lym ide spawn <slug>`).
- A new, empty agent in a window of its own: `lym shuttle open --role <role> --model opus[high]`, which opens an interactive session with no prompt and no run record (tried by hand on 2026-10-30).
- `lym orch distill` and `lym orch refresh` for the orch session.
- The follow toggle of note `viewer-off-shell`.
- "and much else".

## Operator view, not a decision (2026-10-15)

The button set must be highly and easily configurable, not a fixed list in code.

## Placement (operator, 2026-10-30)

The action buttons go on a row of their own, and the `follow` button leaves the row that holds the session names and joins the action buttons.
Reed's status bar has two lines today: line 0 holds the window buttons (overview, agents, shell), line 1 the session buttons, `follow` and the time.
So the action row is a third line, never mixed with the window buttons or the session names.
The operator worried earlier (2026-10-15) that buttons next to others invite misclicks on actions that cannot be undone, such as `lym orch distill`, `lym orch refresh` and ending a run, so the destructive ones need `confirm-before`.

## Open design points

- Ending a run for good (`lym batten cancel <slug>`) gets a CANCEL button of its own, kept apart from HALT, since a halt is reversible and an end is not (operator, 2026-10-30).
  It must be hard to press by mistake: how, for example a tmux `command-prompt` that requires typing the slug, is open.
- Which role and model the new-agent button opens, and whether the role name is asked for or fixed.
- Where the set lives: presumably reed's module config, with a hub default and per-session overrides (the orch session needs other buttons than a task pair).
- What a button entry holds: a label, the command, and which sessions show it.
- How a command runs: `run-shell` in the pair's worktree, with its output and failure visible somewhere.
- Whether the follow toggle is a built-in entry in the same set.

## Operator view, not a decision (2026-10-15)

Prototype the placement by hand in a separate throwaway tmux server, never in a live reed session, before this becomes a task.

## Prototype (folded in from note `reed-status-buttons-prototype`, 2026-10-24)

Operator decision (2026-10-15): a fresh agent builds the prototype when the operator says so, not before.
- A separate throwaway tmux server (its own `-L` socket), never reed's server and never the prime's session `loomyard`.
- A status bar shaped like reed's two lines, with a third line of action buttons (HALT, open in VS Code, a follow toggle), the destructive ones behind `confirm-before`.
- The sample actions only echo or display a message; none runs a real `lym` verb.
- The agent opens the server in a Konsole window for the operator to click through, and tears it down when the operator is done.
- The tmux config used goes into git under `docs/research/` through a docs-only commit, so the chosen layout can seed the task.

## A terminal-window toggle (folded in from note `open-window-toggle`, 2026-10-24)

Whether batten opens a desktop terminal window on a task worktree is decided by `batten.yaml`'s `open_window`, read from the template and the board default layer only, and `lym batten run --window` forces it on whatever that says.
The operator has no quick way to turn the windows off, although each run is a window of the prime's reed session.
Idea: a button in the orch tab's status bar beside `follow` that says whether runs open their terminal window automatically; when it is off, `Open-Window` opens nothing, `--window` included, and the reopen of a resumed run's window too.
The `follow` button keeps its state in the tmux user option `@lym_follow` (`internal/reedengine/bindings.go`), which dies with the tmux server; this switch must survive a restart, so it needs a machine-local home batten can read.
Operator wish, 2026-10-22; the design is open.
