# Card 8 — reedcli-probes-onto-tmuxkit

**Edit:**
- `internal/reedcli/smoke_proctree_test.go`
- `internal/reedcli/smoke_test.go`

**Uses:**
- `internal/testkit/tmuxkit#Pids`
- `internal/testkit/tmuxkit#ProcArgv`
- `internal/testkit/tmuxkit#ProcCwd`
- `internal/testkit/tmuxkit#IsWatchdog`

**Intent:** The second half of D8 as amended: reedcli's own copies of the pid, argv and cwd probes and the watchdog recognizer go, and the kit's are used instead.
`linuxPids`, `linuxProcArgv`, `linuxProcCwd` and `linuxIsWatchdogDaemon` are deleted from `smoke_proctree_test.go`; `linuxProcPPID`, `linuxProcComm`, `linuxDescendantClosure`, `linuxTmuxSocketPids` and `linuxHubHolders` stay and call `tmuxkit.Pids`, `tmuxkit.ProcArgv` and `tmuxkit.ProcCwd`, and the watchdog check becomes `tmuxkit.IsWatchdog(argv)` on an argv read through the kit.
The callers in `smoke_test.go` follow.
The copies in `loomcli`'s `smoke_helpers_test.go` and `orchcli`'s `smoke_live_test.go` stay, per the amendment.
A pure refactor of tmux-tier helpers, covered by the smoke tests that use them.

**ImpactSummary:** Four helpers vanish from one test file; their callers are the sibling helpers in the same package and nothing outside it.

**Verify:** lyx gate test --tags tmux ./internal/reedcli -- -run 'TestSmokeColdStart|TestSmokeLifecycle'
