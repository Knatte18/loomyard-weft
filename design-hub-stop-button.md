# A stop button in the prime orch session that takes the whole hub down, behind a yes/no

`hub-stop-button` · Task · feature, reed, orch, tender

Depends on: [`prime-is-root-beam`](design-prime-is-root-beam.md)

Imported from [issue #122](https://github.com/Knatte18/loomyard/issues/122).

## Seen (operator, 2026-11-11)

To stop everything on a machine without a reboot, the operator has to run several verbs by hand:
`lym beam stop` for the orch and dobby of the prime and of each beam, `lym reed down` in each worktree for the tmux sessions, and `lym tender reap` for run processes no run owns.
The operator finds this cumbersome; it should be one simple action.

## Idea

A button in the reed status bar, shown only in the prime orch's session, labelled "stop" or similar, that takes the whole hub down.
The button asks for a yes/no confirmation before it does anything.

Read first what the button must end: the orch and dobby of the prime and of every beam, the reed sessions of the runs' worktrees, and run processes no run owns.
Decide whether it first pauses the hub, so a run's turn is interrupted cleanly, and what it says it will do in the confirmation.
The operator closes terminal windows (Konsole, the VS Code terminal) by hand.
`machine-restart-shortcut` brings every hub and beam back after a stop or reboot, so the two belong together.

## Placement (operator, 2026-11-15)

This runs as the last member of the beam `adopt-build`, after the adopt redesign's run FSM, push-before-act, Bolt holds and foreground adopt have landed in the beam.
Build the stop on what they deliver: the run FSM and its pause and cancel intents, so a stop leaves every run in a pushed state another start or another machine goes on from.
The adopt decision record kept a hub-wide pause machine-local; this button needs no shared pause between machines either.
