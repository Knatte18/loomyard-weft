# Loom status pane not restored after a reboot and a resumed run

`resume-restores-status-pane` · Task · bug, loom, reed, tender · running · Darn-Write

Imported from [issue #253](https://github.com/Knatte18/loomyard/issues/253).

After the machine rebooted, a task run was resumed with 'lym loom resume' (a Go driver was spawned) and drove on through Drift-Verify, Code-Review, PR-Review and into Finalize. Its loom-status strand stayed dead the whole time ('lym reed list' shows it not live, and the task's tmux session holds only its shell window), so the operator saw no overview pane and took the run for stuck while Finalize ran the slow test tier. Another task resumed by its batten driver after the same reboot got its loom-status pane back. Expected: a resumed run brings its loom-status strand back, or 'lym reed list' or the run's status says the pane is gone and how to restore it.