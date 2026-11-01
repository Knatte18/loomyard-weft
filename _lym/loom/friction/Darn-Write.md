# Running the gate on the darn-write change

The first `lym gate test` printed the tiers' whole stdout, so the tool result was cut in the middle.
The closing line and the one failure were not in the visible part.
To find the failure, I had to look up the board's gate run directory and grep the run's log, which cost two extra turns.
On the next run I sent stdout to a scratch file, and the summary lines were readable.
The installed `lym` predates this task's change, which fixes exactly this.
