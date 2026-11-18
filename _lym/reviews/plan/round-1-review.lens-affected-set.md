```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, key, string or behaviour a card changes, I listed what reads or asserts it and which cards cover that set.

**Per change: the affected set and the cards that cover it**

- **Card 5: `decodeGates` refusal text changes to "want reach or redundancy".** The only assertion is `internal/gateslot/config_test.go:297`. Card 5 updates it. Covered.
- **Card 5: the template's `reports` lines change, `reach` goes first, and `batch` gains `[reach]`.** Readers:
  - `templateDefaults` in `internal/gateslot/config_test.go:70`. Covered by card 5.
  - The comments in `template.yaml` and `doc.go`. Covered by card 5.
  - `internal/loomcli/landingdeps_verify_test.go:33-37`, the `gateConfigWithTwoTiers` replacer, which looks for the exact text `code-burler-fix: {tiers: [0], scope: diff, reports: [redundancy]}`. Its callers are `parity_test.go:633,647`, `overlay_test.go:133`, `recipekeyed_test.go:35` and `landingdeps_verify_test.go:64,81`. **No card covers it.**
  - `internal/gatecli/test_integration_test.go:1499-1528` writes its own config. Not affected.
- **Card 5: three new `Config` keys.** Readers are `ConfigSpec` and `TestLoadConfig`. Covered by card 5.
- **Cards 6 and 14: new slot holders and runs under the redundancy bounds and the memory guard (the reach refresh and `lym gate mutate`).** Readers:
  - The four comments in `template.yaml`. Covered by cards 6 and 14.
  - The memory-guard sentence in `gatecli/doc.go`. Covered by cards 6 and 14.
  - The run list in `PATTERN-test-speed.md`. Covered by cards 6 and 14.
  - `internal/gateslot/doc.go:62-66` (the list of non-tier holders) and `:179-181` (which runs take the three bounds and run fewer tests side by side). **No card covers them.**
  - `docs/overview.md:970,980,995-996` (the memory cap, the build and test bounds, and which runs the memory guard covers). **No card covers them.** Card 6 edits only the overview's gate and "only on command" passages; card 14 edits only its gatecli line.
- **Card 6: the reach step runs before the redundancy step.** `docs/overview.md:954` says "a gate key's advisory `redundancy` report step runs last". Card 6 edits that gate passage. Covered.
- **Card 8: the new `GateEntry.Rerun` field.** Every place that builds or rewraps the entries keeps it by copying: `gatefixer/rowgate.go:86-93,104` and `websterengine/verifygatereport.go:55-57`. The resolver that builds it, `shedrecipe/entries_gate.go:148`, is edited by card 9. Covered.
- **Card 11: four-space triage lines in the failure report.** Readers:
  - `gateslot.FailedTests`. Covered by card 11.
  - `loomshed.failuresSection` and `outputAboveHeader` (`internal/loomshed/verifygate.go`), `runplan.ParseVerifyRun` and weaver's `failureReport`. Covered by card 12.
- **Card 7: the `reach_report` marker.** Readers are `rubricMarkerAllowlist`, shedadapters, shedrecipe and loomcli. Covered by card 7.

**Findings**

1. **MEDIUM, consistency.** Location: card 5, `_lym/plan/05-gate-config-reach.md`.
   - Summary: card 5 rewrites the template's `code-burler-fix` line to `reports: [reach, redundancy]`, but misses `internal/loomcli/landingdeps_verify_test.go#gateConfigWithTwoTiers`. That helper string-replaces the old line exactly. The replacement then silently stops matching, so `code-burler-fix` keeps `tiers: [0]`, which contradicts the helper's doc comment that it lists both tiers. Every test built on the helper (`parity_test.go:633,647`, `overlay_test.go:133`, `recipekeyed_test.go:35`, `landingdeps_verify_test.go:64,81`) then reads a config the helper no longer promises.
   - Evidence: `internal/loomcli/landingdeps_verify_test.go:27-37`, `internal/gateslot/template.yaml:37`.
   - Fix: add `internal/loomcli/landingdeps_verify_test.go#gateConfigWithTwoTiers` to card 5's Edit list and change the matched and replacement lines to the new `reports`.
2. **LOW, consistency (one finding about the method).** Location: cards 6 and 14, `_lym/plan/06-reach-step.md` and `_lym/plan/14-gate-mutate.md`.
   - Summary: both cards cover the doc set naming which runs hold a slot, take the redundancy bounds and run under the memory guard, but only part of it. They update the `template.yaml` comments, the `gatecli/doc.go` memory-guard sentence and `PATTERN-test-speed.md`, and miss:
     - `internal/gateslot/doc.go:62-66`: the non-tier holders, where `lym gate mutate` would be a new one-slot holder.
     - `internal/gateslot/doc.go:179-181`: the runs that take the three bounds and run fewer tests side by side.
     - `docs/overview.md:970,980,995-996`: the memory cap, the bounds and the runs the memory guard covers.
   - Evidence: `internal/gateslot/doc.go:62-66,179-181`, `docs/overview.md:970,980,995`.

**Tool calls**

1. Searched for the old refusal text, readers of the template's `reports`, places that build `GateEntry` and places that rewrap `entry.Gate`, which the manifest lacked.
2. Read the uses of the redundancy report in `loomcli/landingdeps_verify_test.go` and `gatecli/test_integration_test.go`, `loomshed.failuresSection` and `runplan.ParseVerifyRun`, to find the template-line and failure-report readers.
3. Read `gateConfigWithTwoTiers` and `outputAboveHeader`, and searched for tests that read the template text, to confirm the string-replace dependency.
4. Searched for users of `ConfigTemplate()`, `code-burler-fix` in the loomcli tests and literal template lines elsewhere, to find the helper's callers and other copies of the template lines.
5. Read `failedRun.reportLines` and the redundancy mentions in `docs/overview.md`, to check the shape of the failure report lines and the overview's run lists.
