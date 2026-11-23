```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the decision record's Goal and its Added-after-Discussion entries.

**Decisions:**
- **G1 (Goal):** "pr-findings builds its parent-align dependencies itself and records the findings."
  - Built by no change in this diff, which changes no production code.
  - A2 settles it as already fixed on the base by 6fa7f1740, and `_lym/landing/summary.md:12` says so.
  - Not a scope gap.
- **A1 (Added, "Close as overtaken only after a darn-path test"):** asks for a tier-1 test in `internal/loomcli` that drives `lym loom pr-findings` on a darn run through the real cobra tree, and, if it passes on the base, for the change description to say the entry is overtaken.
  - Built by C1 and C2. `summary.md:10-12` covers the change-description half.
- **A2 (Added, "No test seam: the bug was fixed on the base by 6fa7f1740"):** names `TestPRFindingsCLI_SendsBackBlockingFindings` with tapestry and darn rows, and says the entry lands as overtaken with the test as a guard.
  - Built by C1.

**Changes:**
- **C1** (`internal/loomcli/prfindings_integration_test.go:45-77`): renames `TestPRFindingsCLI_SendsBackOnATapestryRun` to `TestPRFindingsCLI_SendsBackBlockingFindings` and makes it a table over the `Tapestry` and `Darn` recipes. Each row runs `pr-findings` through `Command()` and asserts `send_back: true`.
  - Traces to A1 and A2.
  - A1 says "beside" the tapestry test, and C1 adds a table row rather than a second test. A2 names the table test and both rows, so the record itself settles that form.
- **C2** (`internal/loomcli/prfindings_integration_test.go:79-109`): moves the hub setup out of the test into the helper `blockingFindingsRun(t, recipe)` and adds `shedrun.WriteSeed` with the row's recipe (line 90).
  - Forced by C1: a darn row can only exist if the run's seed names darn. `arm.go:105-110` resolves the recipe from the seed and falls back to tapestry when there is none.
  - The tapestry row now writes an explicit tapestry seed where the base relied on the seedless default. That is part of the same forced edit, not new scope.

**Findings:** none. Every change traces to A1 or A2 or is forced by one, and no decision is left unbuilt.

One note outside my lens: the review range has no `Root-Cause:` trailer, and the record is a bug-labelled entry with no plan. The rubric's "Root cause of a bug fix" bullet belongs to the commit-messages check.

Tool calls: none.
