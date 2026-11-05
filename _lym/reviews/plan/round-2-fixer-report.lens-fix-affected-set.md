```yaml
lens: fix-affected-set
outcome: reported
```

Checked whether the fix covers every member of each set finding (F1, F2, F3, F4, F6) by rerunning each finding's search.

**Unchecked:** none. F5 is decision-class, so it is not in this lens's sets.

**F1 — test readers of the members cards 24, 25 and 26 drop:** covered, with one low gap.
- Card 24's readers:
  - `TestLoadConfig`, `seedScopeRecord`, `TestWire_DarnRecipe` and `TestSeatTables` are on card 24.
  - `newBaselineFixture`, `checkRecordedScope` and `TestDriftBaseline` are on card 18.
  - `newVerifyFixture` is on card 16; `recheck_test.go` is deleted by card 23.
  - `driftTestEnv` and `intentionallyNil` are on card 24.
  - `TestDriftDeps_AcceptedSeams…` is deleted on card 24.
- Card 25's readers:
  - `TestLoadConfig`, `TestDriftVerify` and `TestRunCapture` are on card 25.
  - `stageAcceptance` and `TestAcceptWithdraw` are deleted on card 25; `accept_integration_test.go` is deleted on card 22.
  - The `paths.Runs` reads in `TestCapture_OutlivesCaller` sit inside its "compare refuses" steps, which card 22 drops.
  - `compare_test.go:83` is in `TestRunCompare`, whose profile row card 24 drops.
- Card 26's readers: `templateConfig`, `TestSeatTables`, `driftYAML`, `TestDrift_RunsInTheAnchoredDirectory`, `TestModules_EveryTemplateKeyClassified`, `TestDispatchHubWideBoard`, `TestLoadConfig`, `baselineFixture` and `newBaselineFixture` are all listed.

**F3 — literal "Drift-Expect":** covered. After the fix, the only `PastRow(` calls in the plan (cards 8 and 10) use `expectRow`.

**F4 — paths loom's status commit stages:** covered. `statusCommitPathspec` stages the status file, the reviews directory, loom's durable directory, the driver records and the settle records. The run's shed directory is exempt, and the reviews and loom directories fall under the new content rule.

**F6 — overview sentences listing the drift row order:** covered. Line 534, the darn recipe sentence, is now named. Line 617, the regression-rows sentence, and line 616, the Drift-Recheck route, sit in the drift paragraph card 22 already names. The tails on lines 614–615 hold no Drift-Expect position to fix.

**F2 — the current capture's lock wait:** covered. It is the only wait in `CaptureCurrent`.

Findings:

- **LOW** — card 24's removal of `Deps.CompareAccepted`, F1's set.
  - Summary: one reader of `CompareAccepted` is on no card that says it drops that read.
  - Evidence: `internal/shedrecipe/entries_drift_test.go:83`, the `TestDriftEntries` row `{"VerifyNilCompareAccepted", …, func(e *Env) { e.Drift.CompareAccepted = nil }, …}`.
  - Card 16 lists `TestDriftEntries` and stops `driftVerifyEntry` requiring `CompareAccepted`. But its test text names only the rows it adds (the `judgement` gate, `gate_key`, `WeftChangedSince`), not dropping this row.
  - If the row survives card 16, card 24 fails to compile, because `TestDriftEntries` is not in card 24's Edit list.
  - Card 16's test sentence should say `TestDriftEntries` drops its `VerifyNilCompareAccepted` row.
- Out of scope, noted only: card 8's new `expectRow` duplicates the `"Drift-Expect"` literal that card 13's `loomshed.NameDriftExpect` declares.

Tool calls:
1. Grepped `_test.go` files for every member cards 24, 25 and 26 remove — this round's search, which my context lacked.
2. Found the enclosing member of each hit and the Edit lists of cards 18 and 23 — needed to map readers to cards.
3. Grepped card 18 for `checkRecordedScope` and `baseline.md`, and read `capture_integration_test.go:244-290` — to confirm who drops the `paths.Runs` and `baseline.md` reads.
4. Grepped the plan for `PastRow(` and row-name literals, and `docs/overview.md` for row-order sentences — the F3 and F6 sets were not in my context.
