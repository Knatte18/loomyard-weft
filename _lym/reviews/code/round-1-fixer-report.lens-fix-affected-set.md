```yaml
lens: fix-affected-set
outcome: reported
```

Fork task: for F1 and F2, rerun each finding's own search and check the fix covers every member of its set.

**F1 (branches of `checkMarkOf`, `internal/adoptcli/check.go:326-342`).** No findings.
- The finding's premise is the one new `failed` branch with ended tiers and a nil `FailedTier`.
- The added row `FailedAfterAGreenTierWithNoFailedTierNamesBeamStatusAlone` reaches it through `runCheck` with one passed tier and exit 1.
- The gate passed, so the row's expected marks hold.
- The other `failed` branches already have rows: `failed before any tier ended` and `failed at tier1`.
- The `case record.Running != nil` and `starting` branches are covered by the pre-existing rows.
- The `default` branch, running with tiers ended and none in flight, has no row. The diff did not touch it, so it is out of scope.

**F2 (reed verbs under the guard in `internal/reedcli/cli.go:166`).** One finding, `LOW`.
- Changed text: `TestRunCLI_Down_IsNotRefusedInBolt` in `internal/reedcli/cli_test.go`.
- Summary: the fix covers only half of F2's set, and the unguarded verbs are only partly covered.
- Evidence:
  - F2's fix key asks for rows where `attach` or `up` pass the guard in an ordinary task worktree. Neither exists; the fixer dropped them because they start a real tmux server and leaked tmux servers in tier0 and tier2 runs.
  - `up` acceptance is covered live by `mustRunReed(t, prime, "up")` in `internal/reedcli/smoke_debuglog_test.go:59`, a tmux-tagged test, so the gap is real only for `attach`.
  - The verbs defined are `add`, `button`, `list`, `status`, `switch`, `remove`, `attach`, `follow`, `resume`, `up`, `down`, `watchdog` and `window`. Only `attach` and `up` are guarded, because the check at `cli.go:166` matches `cmd.Name()`.
  - The new test covers `down` in Bolt only. The other unguarded verbs have no row, but F2 named only `down`, so this is not a gap against the finding.
  - The fixer must record the `attach` and `up` acceptance rows as deferred in the fixer-report, with that reason.
- Severity: `LOW`. The behavior is guarded by a refusal test and the smoke test, but `attach` acceptance has no coverage.

Calls:
- `grep 'Use:' internal/reedcli/*.go`: listed the reed verbs for F2's set. The context held only `cli.go:145-195`.
