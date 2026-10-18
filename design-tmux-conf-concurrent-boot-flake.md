# Flaky tmux test: TestBoot_TmuxConf TwoWorktreesBootedConcurrently fails with server exited unexpectedly

`tmux-conf-concurrent-boot-flake` · Task · bug, reed · running · Merge-In

Imported from [issue #712](https://github.com/Knatte18/loomyard/issues/712).

Seen 2026-10-17 in run stencilstore-dev-warning-noise, at Publish (publish_verify: go test -tags tmux ./...): internal/reedengine TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently failed with 'run list-commands: exit status 1: server exited unexpectedly'. The task's diff touched only internal/creelcli and cmd/lyx/stencilseedgate_test.go, so the failure is unrelated to the change; batten retried the run. Several runs were in flight on the machine at the time, and publish_verify runs raw go test outside the gate slot pool. Expected: the test boots two tmux servers concurrently without flaking, or isolates them so a concurrent boot cannot kill a server.