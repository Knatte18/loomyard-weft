```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that pins it.

**Per behaviour, the test that pins it**

- **Steps grouped, each group on its own `hubforge.CopyHub` hub, groups run in parallel** (`internal/battencli/lifecycle_integration_test.go:355-424`). This is test-harness structure, not production behaviour. Nothing pins it and nothing needs to; every step function runs exactly as before.
- **Every old step still runs exactly once.** Pinned by the table itself. I checked the old 36-entry list against the new groups: SeedChild 6, FourRowRun 1, TeardownRemote 3, TeardownLocal 6, RunShed 7, CreateRow 7, CommandRows 6. That is 36, each step once, none missing.
- **A failed step ends only its group** (`lifecycle_integration_test.go:419-421`). This is harness control flow with no production behaviour, so it is exempt.
- **No step's assertion goes vacuous under the new grouping.** Each step still sets up its own precondition on its group's hub:
  - Every step either creates its pair itself with `AddPair`, `seedBoardTask` or a create step, or, like `stepRunRowCommand_ReportsATimeoutAndAnExitCode` (`:1560`), reads no hub state at all.
  - `stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent` (`:1806`) sets the orch strand before every row.
  - The tracked-layer, switch and resume Command-row steps each write their own board commands through `newCommandRowFixture` (`:1313`).
  - `stepDirtyPrime_CreateRowBlocksBeforeAnythingCreated` dirties the prime itself (`:1262`).

  No step's assertion depends on state that a step now in another group used to leave behind.
- **The start-cost wall-clock bound** (`:1497`, `:1547`). It is still asserted by `TestBattenIntegration_Rows/CommandRows/StartCost_CommandRowsCostUnderTheBoundBeyondTheCommitAndTheProcess`. Neither the bound nor the comparison changed, so no check was loosened.
- **Assertions kept.** The diff touches only the table and the test's doc comment. No step body changed, so no assertion was widened or removed.
- **`pattern/PATTERN-pair-remote.md:140` renamed subtest path.** This is static prose in a doc, so it is exempt. The path it names, `TestBattenIntegration_Rows/CreateRow/RecordsPrimeRefusal`, matches the group and step names at `lifecycle_integration_test.go:392` and `:398`.

**Findings**

None. The diff adds no production behaviour, so no coverage is missing. It adds no new test that duplicates existing coverage, it loosens no check, and no test is tautological.

Tool calls: none.
