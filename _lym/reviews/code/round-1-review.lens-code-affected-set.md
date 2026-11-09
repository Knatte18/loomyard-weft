```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. No findings.

**Per change, the set found and whether it is covered:**
- **`GateKey.Reports` and `ReportRedundancy`:**
  - `Gate()` clones `Reports` (`tiers.go:357`), `decodeGates` validates it, and the only consumer is `run.go` (the `slices.Contains` check). Covered.
  - `Lints` is read only at `config.go:494` and `run.go:193`, so no other site copies `GateKey` fields and misses `Reports`.
  - The template, `gateslot/doc.go`, `gatecli/doc.go`, `docs/overview.md` and the config tests are updated.
  - The `landingdeps_verify_test.go` string replacer is updated to match the new template line.
- **New keys `redundancy_max_tests` and `redundancy_budget_min`:**
  - They are in `configFile`, `ConfigSpec` (cross-pair), the `decodeConfig` bound check, `Config` and `template.yaml`.
  - They are documented in `gateslot/doc.go`, and a `LoadConfig` test row pins each ≥1 bound.
  - A repo-wide grep for the existing sibling key `redundancy_test_timeout_sec` finds no other key list or doc that would need the new keys.
  - The two existing mentions of the timeout keys (`docs/overview.md:906`, `gatecli/doc.go:553`) describe the testmap verb, not the step, so they stay correct.
- **`testRequest.secondUnit`:** set at both construction sites (`cli.go`, `rungate.go`) and read in `redundancy.go`. `testmap.go:57` has its own `secondUnit` on a separate struct. Covered.
- **`TestFunc.File` and `ParseTestFuncs`:** the only other `TestFunc{` literal is `testfuncs_test.go`, which the diff updates, and `testcover/doc.go` is updated. Covered.
- **`Judge` to `JudgeAgainst`:** `Judge` delegates with nil `kept`, so existing callers keep their signature. I read no caller and ran no test, so "callers unaffected" is by signature only. The `verdict_test.go` rows are the code-tests lens's.
- **`PATTERN-gate-slot` non-tier holder list:** it points at `gateslot/doc.go`, and the diff adds the step there. Covered.
- **Declared PATTERN changes:** neither the decision record nor the focus declares any, and the diff has no `pattern/` or `PATTERN.md` change. Nothing is owed.
- **Commit messages and departures:** no `Pattern-Departure:` trailers, and none are needed.
- **Stencils:** the diff changes none.
- **Not covered:** `PATTERN-test-speed`'s "Gate keys" table lists no reports column. That is optional, since the table only names site and scope, so I raise no finding.

**Tool calls (1):** one combined grep/diff-stat over the repo. It checked other readers of the redundancy keys, other `Lints`/`GateKey` copy sites, other `TestFunc` users and any pattern/doc diff, none of which my context held.
