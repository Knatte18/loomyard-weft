---
round: 1
focus:
  - "Card 1, test economy: `TestParseSeed` plans cases (retired `parent` key, absent driver defaulting to `DriverGo`) that `TestSeed_LegacyParentKey` and `TestReadSeed_AbsentDriverDefaultsToGo` in `internal/shedrun/seed_test.go` may already cover through `ReadSeed`. Settle which `TestParseSeed` rows are new behavior and which repeat an existing test."
  - "Card 2, granularity: the Create list names `turnPoint`, `endsTurn`, `workingIntervals` and `tableTime` as separate symbols. Settle which are private supporting types or helpers that belong inside another symbol's card entry rather than a glyph of their own."
  - "Card 4, reachability of refusals: the decode and timestamp-parse aborts and the `ErrPathNotAtRevision`, empty-history and no-tag returns each need a named test input that reaches them. Check that the planned fake-`ArchiveTrees` cases reach each branch, including a bad `last_at` and a bad seed `started_at`."
  - "Card 5, reachability and upstream rejection: the `resolveHub` refusal and the `os.Stat` non-not-exist abort need a named input that reaches them and that no earlier check rejects. Check `TestRunRefuses` and `TestResolveHub` against `lyxcwd.Getwd` and `RecordsWorktree` failing first."
  - "Fidelity to the decision record: confirm each Decision (1 to 10) and each acceptance criterion is carried by some card, including the 'All runs' sums with no cross-run union, the `<role>+sub` rows, the report's explanatory paragraph, the newest-tag choice and the seed-without-`started_at` bounds. Confirm no card adds scope the record does not license, for example card 5's calibration reuse of the `gitrepo.Repo`."
  - "Card order and verify: check no card uses a symbol a later card creates (card 4 uses card 2's `covered` and card 1's shedrun symbols, card 5 uses card 4's). Check that `## verify:` runs the tests of every targeted package, `internal/shedrun`, `tools/tokencount` and the integration-tagged `time_integration_test.go`, and that its order matches the plan template."
  - "Test budget: cards 1, 2, 4 and 5 each list `cmd/lyx/testdata/test-budget.yaml` under `Edit:`. Check each raise matches that card's new top-level tests, and that card 3, which edits `count_test.go`, adds no new top-level test."
---

The focus lists are drawn from the plan and the decision record only.
No `Do not flag` item is promoted: dependency fields, `ImpactSummary` on non-`Edit` cards and the gate's mechanical checks are left out.
