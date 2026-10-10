# Card 2 — poll-floors

**Create:**
- `plan:internal/orchengine#pollFloor` -> `const pollFloor = time.Second`
- `plan:internal/shuttleengine#pollFloor` -> `const pollFloor = time.Second`
- `plan:internal/battenshed#pollFloor` -> `const pollFloor = time.Second`

**Edit:**
- `internal/orchengine#Config.PollInterval`
- `internal/orchengine#LoadConfig`
- `internal/orchengine#defaultPollIntervalMS`
- `internal/orchengine/template.yaml`
- `internal/orchengine/config_test.go`
- `internal/shuttleengine#pollInterval`
- `internal/shuttleengine#defaultPollIntervalMS`
- `internal/shuttleengine#LoadConfig`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/config_test.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/fixture_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/gate_list_test.go`
- `internal/shuttleengine/gate_test.go`
- `internal/shuttleengine/claudeengine/corpus_test.go`
- `internal/shuttlecli/cli_test.go`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed/innerrun_test.go`
- `internal/shedrecipe/entries_batten_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Floor every configured poll interval at one second, floored not refused (D9).
`orchengine.Config.PollInterval` returns `pollFloor` for any value below it, non-positive included, in place of the template default; `LoadConfig` logs one Warn naming `poll_interval_ms` and the floor when the loaded value is below it, and the accessor floors silently, so a daemon that reads the accessor every tick logs once.
`template.yaml`'s `poll_interval_ms` comment says a value below 1000 is floored to 1000 with one warning.
`TestConfig_AccessorsFloorNonPositive` expects one second for `PollInterval`, and `TestLoadConfig_PresentFileOverrides` gains the Warn assertion over `logcapture`, together with a row proving a value at or above the floor passes through unwarned; the production timing assertion D2 asks for is that `defaultPollIntervalMS` is at or above 1000.
`shuttleengine.pollInterval` floors a `PollIntervalMS` below 1000 to `pollFloor`, `defaultPollIntervalMS` becomes 1000, `template.yaml`'s default becomes `poll_interval_ms: 1000` with its comment reworded to the floor rule, `liveness_every_n_polls` stays at 10, and `LoadConfig` logs the one Warn as orch does; `awaitStartup`'s comment names the new cadence.
`TestPollInterval_FloorsNonPositive` gains the sub-second row and the floor value, and `config_test.go` repins the template default and seeds a value at or above the floor in the module-threading test.
Every shuttle test fixture that configures a sub-second poll interval to run fast (`fastConfig`, `shortStartupConfig`, `sparseProbeConfig`, `gateConfig`, the inline configs in `startup_test.go`, `attach_test.go`, `gate_list_test.go`, `gate_test.go`, `corpus_test.go` and `shuttlecli/cli_test.go`) keeps its speed through an injected clock, never a config value: `newFixture` defaults to a fake clock whose `Sleep` advances it, a test that needs the wall clock says so with `withClock`, and `shuttlecli`'s runner tests install one through `Runner.SetClock`; tick-count expectations that depended on the old interval are repinned.
`battenshed.NewInnerRun` floors a told `pollInterval` below one second to `pollFloor` with one Warn naming `poll_interval_s` and the floor, so the recipe's `poll_interval_s` can never drive a sub-second wait; `innerrun_test.go` asserts the floor and the Warn on a sub-second duration and the pass-through of one second, and `entries_batten_test.go` asserts `defaultInnerRunPollIntervalS` yields at least one second.
The recipe entry keeps resolving an absent or zero `poll_interval_s` to its default without a warning, since the two are indistinguishable there.

**ImpactSummary:** Three accessors change their floor and shuttle's template default doubles; every shuttle test that configured a sub-second poll moves to a fake clock, and no caller signature changes.
