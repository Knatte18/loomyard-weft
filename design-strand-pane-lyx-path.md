# strand pane resolves bare lyx to the production binary while LYX_BIN names the dev build

`strand-pane-lyx-path` · Note · bug, reed, shuttle

Imported from [issue #680](https://github.com/Knatte18/loomyard/issues/680).

Seen: in a sandbox strand pane, `command -v lyx` names `/home/knatte/go/bin/lyx` while `LYX_BIN` names `.dev-bin/lyx`. The pane PATH begins `/home/knatte/go/bin:` because `~/.bashrc` prepends it after reed sets PATH, so a Claude strand that runs bare `lyx creel read` talks to the sandbox daemon through the production binary. The shuttle sandbox suite pre-condition (`command -v lyx` names the spawning binary) fails on this machine.

Expected: bare `lyx` in a strand pane resolves to the spawning binary, as the suite pre-condition states, even when the shell rc prepends another directory.

Found by the creel stage 1 re-test of 2026-10-16.