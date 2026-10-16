# reed down ends its watchdog; strand panes resolve lyx to the spawning binary

`reed-down-and-pane-path` · Task · bug, reed, shuttle · running · Merge-Burler-Fix

Two reed defects from the creel stage 1 re-test of 2026-10-16.
Operator decision 2026-10-16: the two run as one low-budget task.

## reed watchdog outlives lyx reed down (#679)

Imported from [issue #679](https://github.com/Knatte18/loomyard/issues/679).

Seen: in the sandbox hub, after `lyx reed down`, the reed watchdog process was still running; after the dev binary was redeployed it respawned on the new binary, and it had to be killed by pid.

Expected: `lyx reed down` ends the watchdog of that worktree reed session, so no reed process remains after it.

Found by the creel stage 1 re-test of 2026-10-16 (teardown).
## strand pane resolves bare lyx to the production binary (#680)

Imported from [issue #680](https://github.com/Knatte18/loomyard/issues/680).

Seen: in a sandbox strand pane, `command -v lyx` names `/home/knatte/go/bin/lyx` while `LYX_BIN` names `.dev-bin/lyx`. The pane PATH begins `/home/knatte/go/bin:` because `~/.bashrc` prepends it after reed sets PATH, so a Claude strand that runs bare `lyx creel read` talks to the sandbox daemon through the production binary. The shuttle sandbox suite pre-condition (`command -v lyx` names the spawning binary) fails on this machine.

Expected: bare `lyx` in a strand pane resolves to the spawning binary, as the suite pre-condition states, even when the shell rc prepends another directory.

Found by the creel stage 1 re-test of 2026-10-16.