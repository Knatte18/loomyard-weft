# lyx batten run starts in its own window by default

`batten-window-default` · Note · enhancement, batten, reed, orch

## Seen

The orch starts every run with `lyx batten run <slug> --window`, and the operator confirms that this is always how runs are started.
Without the flag the whole lifecycle runs in the caller's foreground for hours, which blocks the orch session and can be killed by the shell timeout.

## Idea

Make the window the default for `lyx batten run`: start the run in its own tmux window of the prime's reed session, return at once, and run the Open-Window row.
Keep the foreground as an explicit opt-in flag for the cases that need it, such as the window's own inner invocation and tests.
Update the orch role and the `lyx batten run` help to drop `--window` from the normal path.