```yaml
lens: code-tests
outcome: reported
```

List per behaviour (lens: code-tests):

- **Passing `--card` call prints the not-run line naming every declared tier no listed tier is or covers.** Pinned by `scopes_integration_test.go:507-519`, the `--gate with scope card runs the --card packages` row. It asserts the exact line `gate test: tier1, tier2, tier3, tier5, tier6, tier7 did not run, so this pass is not green for them\n`, and a wrong set or wording would fail it. The base config is `scenarioConfig` plus tier1 covering tier0.
- **The tier set is the declared tiers minus the listed tiers and the tiers they cover.** The row at `scopes_integration_test.go:519-524` pins the covered-tier drop with `--card 1 --tier 1`, which asserts `tier2, tier3, tier5, tier6, tier7` with no tier0. Pinned.
- **The line is silent under other scope forms.** The `--base` row asserts no line at `scopes_integration_test.go:185`, but that assertion sits in the "a computed scope under go…" row at 176-192, not the `--base` row at 347. The explicit-target row (548) and the `--all` row (527) have no assertion, and neither does the `--gate diff` row (539). Only the computed form is pinned. A regression that prints the line under `--base`, `--all` or explicit targets passes the suite. Finding 1.
- **A failing `--card` call prints no line.** Pinned by `scopes_integration_test.go:455-457`, in the row that fails a card on a tier1 vet run, with `strings.Contains(output, notRunSuffix)`. Covers the code == 0 guard. A halted call (D3) has no assertion, but the halt path returns before the line, so the gap is minor.
- **A vet run under a tier's tag does not count as running that tier.** No row pins it. The vet row at 431-439 is a passing `--card 1` call with vet runs under tier1, and it does not assert the line. The row at 507 plans no vet run. Dropping `!run.vetStep` from `notRunTiers` therefore leaves every test green, because `--card` with only tier0 listed would then count tier1 as run when it was only vetted. Finding 2.
- **A guard run under a tier does not count as running that tier.** No row pins it. The guard group in these rows belongs to tier0, which is listed anyway, so removing `!run.guardStep` changes nothing. This is low risk.
- **A manual tier is counted among the not-run tiers.** No row pins it, and no `manual` setting appears in the scopes config. `notRunTiers` has no manual-specific branch, so the behaviour follows from the declared-tier iteration. Remove if you count that as no separate behaviour. Finding 3.
- **The line is informational: exit code stays 0 and nothing is recorded.** Pinned only by exit 0 at `scopes_integration_test.go:521`. No row asserts that the green record is unchanged. The code adds no write, so this is low risk.
- **Line goes to stderr.** The harness merges stderr into stdout through `clihelp.ExecuteIn`, so the destination is unpinned, like every other summary line in this file. Not a finding.
- **`doc.go` and the spec bullet.** Static prose, exempt.

Findings:

1. LOW, class `design` (coverage gap). `internal/gatecli/scopes_integration_test.go:527-556`. The D2 rows for `--all`, explicit targets (`--gate card ./other`) and `--gate diff` carry no not-run-line assertion. Only the computed row at 185 does. The `sc.form == scopeCard` guard at `internal/gatecli/run.go:173` is pinned in one direction only. Evidence: the diff adds assertions at hunks 185, 455, 511 and 519 only.
2. LOW, class `design`. `internal/gatecli/tiers.go:420-425`, the `!run.vetStep` and `!run.guardStep` exclusions in `notRunTiers`, which D1 requires ("a vet run under a tier's tag … does not count as running the tier"). No test row would fail if they were broken. The vet row at `scopes_integration_test.go:431-439` is the natural place for an assertion naming tier1 among the not-run tiers.
3. NIT, class `design`. A manual tier among the declared tiers (decision record auto-mode assumption) has no row. The config in `scopes_integration_test.go:37-50` declares none.

Tool calls (4 of 6):
1. `grep` over `scopes_integration_test.go` for `t.Run`, `func` and `stderr`: to map the diff's assertions to subtests, which the manifest lacked.
2. `sed` over `test_integration_test.go:195-232`, plus the `scopes_integration_test.go` rows at 388-402, 503-520 and 540-556: to see the harness's stdout and stderr wiring and the unchanged rows.
3. `grep` for `printLine`, `stderr:` and `SetErr`, plus `sed` over `test_integration_test.go:95-135`: to confirm the harness routes stderr to stdout, so the assertions on `g.stdout` are valid.
4. `sed` over `scopes_integration_test.go:30-52` and 431-441, plus `grep` for manual and covers settings: to confirm the config (tier1 covers tier0, no manual tier) and that the vet row asserts no line.
