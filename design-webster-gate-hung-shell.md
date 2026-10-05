# Webster verify gate waits forever when Master ends its turn with a hung background shell

`webster-gate-hung-shell` · Note · bug, webster

Imported from [issue #361](https://github.com/Knatte18/loomyard/issues/361).

Seen 2026-10-04 in `board-model`: after the verify gate re-prompted Master at 17:07:33, Master's fixer fork fixed the failure and Master's turn ended at 17:11, but the step logged nothing more for 75 minutes.
Master's Stop events (`.lyx/shuttle/<guid>/events.jsonl`) all carried `background_tasks` with a shell in state `running`: `python3 - 2>/dev/null; cat > /dev/null <<'EOF' … EOF; grep …`, where `python3 -` sat reading stdin forever.
The gate apparently does not count a Stop with a running background task as a turn end, so one hung shell blocks the run indefinitely; an operator prompt at 18:25 produced another such Stop and changed nothing.
Killing the `python3` process let the gate rerun verify at 18:26.

Expected: a turn end whose only running background task is a shell (not a fork) still counts, or the gate bounds the wait and surfaces "turn ended with background task X running" to the driver, which today cannot see or stop it (ly-drive has no verb for it, and Webster's interrupt policy is handback).
Also: lyx agents should not run `python3` at all (operator rule); the agent settings shuttle writes could deny it.
