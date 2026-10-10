# Fold loomcli and orchcli tmux probe copies onto tmuxkit

`tmuxkit-probe-copies` · Note · enhancement, loom, orch, reed

From `test-suite-weight` Discussion review (2026-10-10, round 3 F2): its D8 moves `reedcli`'s watchdog recognizer and `/proc` probes into `tmuxkit`, but `loomcli` (`smoke_helpers_test.go`) and `orchcli` (`smoke_live_test.go`) keep their own copies.
A parent decision kept those two copies out of that run's scope.

Idea: fold the `loomcli` and `orchcli` copies onto `tmuxkit` once `test-suite-weight` lands, per `PATTERN-testkit`.
