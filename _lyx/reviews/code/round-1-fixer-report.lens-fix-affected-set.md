```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F2 and F5 fixes cover every member of their sets. The other findings' premises are not sets, so I left them unchecked.

No findings.

- **F2** (fixtures that build a shuttle config from the shipped template, so they inherit `start_retries: 1`): `providerlessShuttleConfig` has two call sites, `smoke_helpers_test.go:72` and `smoke_helpers_test.go:254`, and both now get `start_retries: 0`.
- Two other templates in `loomcli` are rewritten independently: `driverShuttleConfig` (`smoke_driverstrand_test.go:116`) and `integrationDriverShuttleConfig` (`integration_driverbootstrap_test.go:89`). Both use a stubbed provider that reaches ready, so they should not hit a died start. I read their comments only.
- `orchcli` (`smoke_live_test.go:92`, `smoke_cycle_test.go:177`), `webstercli` (`verbs_test.go:1363`) and `treadleengine` (`smoke_judge_test.go:223`) use the plain template. I did not check whether their providers can die at start.
- **F5** (callers of the stdin-carrying exec): the only production caller of `runStdin` with non-nil stdin is `io.go:150`, which goes through the same `runStdin` the new row drives. No other site needs a row.
- Unchecked: F3 and F4. Neither premise is a set I could rerun a search for.

Tool calls:
1. grep of the `loomcli` and other test packages for `providerlessShuttleConfig`, `ConfigTemplate()` and `start_retries`, because my context did not show who else builds a shuttle config.
2. grep of non-test files for `runStdin(` callers, because my context did not show them.
3. `sed -n` of the `smoke_driverstrand_test.go` and `integration_driverbootstrap_test.go` config helpers, to see whether those fixtures use a died-start provider.
