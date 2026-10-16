# reed watchdog outlives lyx reed down and respawns on a redeployed binary

`reed-watchdog-outlives-down` · Note · bug, reed

Imported from [issue #679](https://github.com/Knatte18/loomyard/issues/679).

Seen: in the sandbox hub, after `lyx reed down`, the reed watchdog process was still running; after the dev binary was redeployed it respawned on the new binary, and it had to be killed by pid.

Expected: `lyx reed down` ends the watchdog of that worktree reed session, so no reed process remains after it.

Found by the creel stage 1 re-test of 2026-10-16 (teardown).