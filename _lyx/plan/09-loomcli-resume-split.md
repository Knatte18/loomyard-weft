# Card 9 — loomcli-resume-split

**Create:**
- `internal/loomcli/smoke_resumearming_test.go`

**Edit:**
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D6's consequence, ahead of the runtime check card 10 adds: tier 2 starts no tmux server.
`TestLoomResumeReconcilesTheHubConfigBeforeArming` today ends with a `loom resume` that arms reed and so starts the hub's tmux server inside an integration-only build.
Its reconcile half stays in the integration file: the pair is seeded with a reed config that refuses at `Up`, the way `newBadReedUpFixture` seeds one, so `loom status` still leaves the retired key and writes no stamp, `loom resume` on the broken `loom.yaml` still refuses naming the file and the way forward, and `loom resume` after the fix reconciles the hub config, commits it and writes the stamp, then refuses at reed `Up` before any server starts; the test asserts no watchdog pid and no driver pid afterwards, as `TestLoomFailedReedUpRefuses` does.
Its arming half moves to the new `tmux`-tagged file as `TestLoomResumeArmsAfterReconcile`: a good reed config, the same retired key and stamp fixture, `loom resume`, and the assertions that the stamp exists and the pair's session is up on the hub's key, registered through `tmuxkit.KillOnCleanup`.
The file's header names the reason for the split.
`loomcli`'s budget row rises by one.

**ImpactSummary:** One integration test loses its arming tail; no production code changes.

**Verify:** lyx gate test --tags integration ./internal/loomcli -- -run TestLoomResumeReconcilesTheHubConfigBeforeArming
lyx gate test --tags tmux ./internal/loomcli -- -run TestLoomResumeArmsAfterReconcile
