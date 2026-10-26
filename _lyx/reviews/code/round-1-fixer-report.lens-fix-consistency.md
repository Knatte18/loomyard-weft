```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I found one `LOW` finding and nothing at `MEDIUM` or above.

- **LOW** — `providerlessShuttleConfig`, the `start_retries: 1` → `0` rewrite in `internal/loomcli/smoke_helpers_test.go`.
  - The rewrite ignores a miss, so if the template line drifts the 5 s retry returns without any failure.
  - Its twins disagree on drift: `claudeConfigWithBinary` panics and `driverShuttleConfig` calls `t.Fatalf`.
  - The `startup_timeout_s: 90` rewrite next to it is also silent, so the new line does match its immediate neighbour.
  - Evidence: `internal/loomcli/smoke_driverstrand_test.go:117-120` asserts `strings.Contains` before replacing.

Checked, no mismatch:
- **Other shuttle-template users:** `driverShuttleConfig` and `integrationDriverShuttleConfig` run a real stub provider that reaches ready, so no died start occurs and no retry is needed. The only non-stub user of `providerlessBinary` is `providerlessShuttleConfig`'s own fixtures; `smoke_starttail`, `smoke_driverstrand` and `integration_driverbootstrap` use stubs.
- **Orch and webster smoke fixtures:** `internal/orchcli/smoke_live_test.go:92`, `internal/orchcli/smoke_cycle_test.go:177` and `internal/webstercli/verbs_test.go:1363` use the unmodified template. I did not open them; if any of them seeds a providerless binary and is meant to stay bounded, it would still retry once.
- **`keyTimesReed` doc:** its comment now sits directly above the type, and the test comment starts at `TestStartup_DiedStartLaunchesAgain`.
- **`wantRemoveWaits` rows:** the removal waits asserted in `TestStartup_DiedStartLaunchesAgain` and `TestStartup_PaneNotLiveMidStartup` match `startupRemoveFirstDelay`, doubled, in `removeStartupStrand` (`wait.go:802-814`).
- **Timeout row:** it uses `StartupTimeoutS: 3600` because a startup-window expiry classifies as died (`wait.go:725`), and only the run-deadline path gives `OutcomeTimeout` (`wait.go:708`). Its row comment says why.
- **Template comment:** `template.yaml`'s "a timed-out start … never retried" reads loosely against that split, but the fixer did not change it.

Tool calls:
1. grep for other shuttle-template users in tests, to find unretired twins of the smoke helper.
2. sed of `smoke_driverstrand_test.go` and `integration_driverbootstrap_test.go`, to see how those twins rewrite the template (drift check, real stub provider).
3. grep for `providerlessBinary` and `claudeConfigWithBinary` users, to confirm which fixtures start a died provider.
