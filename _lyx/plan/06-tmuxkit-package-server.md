# Card 6 — tmuxkit-package-server

**Create:**
- `plan:internal/testkit/tmuxkit#PackageServer` -> `func PackageServer(t *testing.T, tmux string) string`
- `plan:internal/testkit/tmuxkit#packageServerOnce` -> `var packageServerOnce sync.Once`
- `plan:internal/testkit/tmuxkit#packageServerKey` -> `var packageServerKey string`

**Edit:**
- `internal/reedengine/contract_integration_test.go`
- `internal/testkit/tmuxkit/tmuxkit.go`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/reedcli/smoke_coldstart_test.go`
- `internal/reedcli/smoke_debuglog_test.go`
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_lifecycle_test.go`
- `internal/reedcli/smoke_resume_test.go`
- `internal/reedcli/smoke_selvage_keepalive_test.go`
- `internal/reedcli/smoke_staterecovery_test.go`
- `internal/reedcli/smoke_switchseed_test.go`
- `internal/reedcli/smoke_teardown_test.go`
- `internal/reedcli/smoke_warmpath_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/loomcli/smoke_driverstrand_test.go`
- `internal/loomcli/smoke_drivingsurface_test.go`
- `internal/loomcli/smoke_gate_test.go`
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/loomcli/smoke_starttail_test.go`
- `internal/loomcli/integration_driverbootstrap_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_adopt_large_test.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/webstercli/cli_tmux_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/shuttlecli/smoke_guardrail_test.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/shuttlecli/smoke_run_test.go`
- `internal/shuttlecli/smoke_turns_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/testkit/tmuxkit#startServer`
- `internal/testkit/tmuxkit#registerKey`
- `internal/testkit/tmuxkit#KillOnCleanup`
- `internal/reedengine#ServerName`

**Prosa:**
- `pattern/PATTERN-test-isolation.md`

**Intent:** D5: one server per package for the tests that name their own sessions.
`PackageServer` mints one key per test binary under `packageServerOnce`, starts it through `startServer` and registers it; it registers no cleanup, because `Main`'s sweep kills every server under the kit's directory.
A test qualifies when it names its own sessions, kills no server and asserts nothing about the server's whole session set; a test that drives reed through a hub keeps the hub's `reedengine.ServerName` key, and a test that runs `Down` or `kill-server`, or compares `list-sessions` to an exact set, keeps `Socket`.
Classified onto `PackageServer`, all in `contract_integration_test.go`: `TestExactSessionTargetsNeverPrefixMatchSiblings`, `TestDisplayMessageDoesNotErrorForAnAbsentSession` and `TestSessionNameRewriteIsSilentAndExactTargetsMissIt`; each kills at most its own sessions and checks its own names by containment.
Every other tmux-tier test keeps its key: `TestMultiplexerContract` and `TestSelvageNeverGetsZeroHeightLayoutCell` compare the session set or run `Down`, and the reedcli smoke tests bring the server down.
A kept key reaches D4's hermetic server only when it is registered before reed's first boot on it, so every `tmux`- and `llm`-tier test that boots reed registers its key through `tmuxkit.KillOnCleanup` at the point the key is known and before the first `Up`, `EnsureSession`, `Resume` or `lyx` run that arms reed: `reedengine.ServerName(h.Path)` right after the hub is built, and for `webstercli`'s standalone session the `lyx-<hash8>` key `standalonestate.Derive` yields.
The registration sits in the `tmux`- or `llm`-constrained file itself, or in a helper only such files compile; a helper shared with an `integration` build, such as loomcli's `smoke_helpers_test.go`, never calls tmuxkit, since that would start a server in tier 2.
reedcli's `socketAndSessionIn` today registers after `reed status`, after the boot, so the listed reedcli smoke files register right after their hub is built and the late call stays only as a no-op.
The one exception is `smoke_debuglog_test.go`: `TestSmokeDebugLog` asserts that reed's own server spawn, run with `-v` from the hub logs dir, writes `tmux-server-*.log` there, which a pre-started server never does.
It registers right after its first `up` instead of before it, so every server it observes is reed-started and carries card 5's bound for servers reed starts itself.
`reedengine`'s integration helpers (`newIntegrationEngine`, `newColdScratchEngine`) and reedcli's `skipWithoutMultiplexer` already register before boot and are unchanged.
The listed files are every `tmux`- or `llm`-constrained test file that boots reed without such a registration.
For the `tmux`-tier packages, card 10's `tmux` Verify run, which runs the tier for `reedcli`, `loomcli` and `reedengine` and the other reed-booting `tmux` packages, is the net for a missed one: its check fails a tagged package on an unregistered socket still present at the end of the run.
The `llm`-tier packages are only compiled before Publish, and Publish does not run that tier, so a missed registration there is left to this census and review.
`tmuxkit_integration_test.go` gains `TestPackageServer_OneKeyPerBinary`: two calls return the same key, the server answers `list-sessions` on it, and the key is registered.
`tmuxkit`'s budget row rises by one.
`tmuxkit`'s package comment gains the `PackageServer` sentence; `PATTERN-test-isolation.md`'s tmux socket files section gains the `PackageServer` sentence, the classification criterion and the register-before-boot rule.

**ImpactSummary:** Three reedengine tests change their key source and every reed-booting tmux- and llm-tier test file gains one registration line before its first boot; no production code changes.

**Verify:** lyx gate test --tags tmux ./internal/testkit/tmuxkit ./internal/reedengine
lyx gate test --tags tmux ./internal/reedcli -- -run 'TestSmokeDebugLog|TestSmokeColdStart'
lyx gate test --tags tmux ./internal/reedcli ./internal/loomcli ./internal/pairteardown ./internal/webstercli -- -run '^$'
lyx gate test --tags llm ./internal/orchcli ./internal/burlerengine ./internal/seatengine ./internal/shuttlecli ./internal/treadleengine -- -run '^$'
