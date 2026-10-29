# Support log — creel-flakes-and-adopt-names

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- Parse `suite.log` after each tier for `--- FAIL` lines: the log accumulates every tier's text, so each tier would need its own offset, and the counter already reads the structured events.
- A failures list kept only on the observer call, outside the run record: it would add a second channel beside the record the observer already receives.
- Fix the lifecycle flake in the daemon's shutdown ordering: the lock is released before `Run` returns, so no production change closes the race; the race is in the test's sync.
- Keep counting idle reads in the hook-wait subtest but drain them first: it still rests on how many reads a post causes, which no signal pins.
- Raise the shared fake clock's ten-second real-time bounds: they only bound a failing test, and the EXTERNAL failures came from stale counts, not from the bounds.

## Review rounds

_No rounds yet._

## Question ledger

- Should the per-tier mail list every failing test? Self-pick: at most ten, then `and <n> more`; `suite.log` and `check.json` hold the rest.
- Should a failing parent test be listed beside its failing subtest? Self-pick: leaves only, since the parent's failure carries no line of its own.
- Does the lifecycle flake call for a retry of the daemon start? Self-pick: no; the second daemon was the waiter's own spawn, so the test now proves the waiter connected before it stops the first daemon and gives the waiters no spawn seam.
