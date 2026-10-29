# Flaky creel daemon lifecycle test: 'another daemon holds this hub' on restart

`creel-daemon-restart-flake` · Note · bug, creel

Imported from [issue #895](https://github.com/Knatte18/loomyard/issues/895).

A gate run failed once on the creelengine `TestDaemon_Lifecycle` subtest covering a hook wait that reconnects across a daemon restart.
The failure text was "the daemon ended before it listened: creel: another daemon holds this hub" (`server_test.go`).
The change under test touched only prose and doc comments, and the identical gate passed on the next run and on the batch gate.

This is abnormal: a passing-then-failing test with no code change points to a race between the first daemon releasing the hub and the second daemon starting.
Look at the hub-release ordering in the creel daemon shutdown path and make the test wait for the release (or retry the start) so it is deterministic.

Provenance: task `creel-hook-post`, note `15-creel-post-interrupt.md`.
