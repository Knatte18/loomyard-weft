```yaml
lens: code-tests
outcome: reported
```

Restating my task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `withTimeoutMin` ends the context once the bound passes, and polls at most every `timeoutMinPoll` or what is left of the bound: `TestWithTimeoutMin`.
2. `withTimeoutMin` counts the wall clock when the monotonic clock fell behind across a host sleep: **none** (F1).
3. The tier timeout is told apart by the `errTimeoutMinPassed` cause (`tierrun.go:166`): `TestRunTierKillsWhatLeftTheProcessGroup`, row `ACommandThatOutlivesTheTimeout`, which asserts `TimedOut`.
4. The build timeout is told apart by the same cause (`build.go:61`): `TestRunBuildTimeout`.
5. `runTiedChild` and `killMarkedLeftovers` kill a process that left the group, both after a normal exit and after a timeout: `TestRunTierKillsWhatLeftTheProcessGroup`, both rows.
6. A nested gate's marker stays unique, and the `cmd.Env == nil` branch uses `os.Environ()`: none. This is low value, so I filed no finding.
7. `observeSummary` sums the multi-line `Total tests:` block per outcome: `TestCounters`, row "a detailed console logger's summary of several lines…".
8. The block ends at a line that is not an outcome line: the same row, at its `Total time:` line.
9. `startInvocation` and `replaceInvocation` reset `inTotals`: `TestVSTestCounterInvocationClosesAnOpenSummary`.
10. `observeFailure` closes a failed test's block at a `Total tests:` line: **none** (F4).
11. `renderTierCommand` and `takeCallerFilters` handle the `--filter X`, `=X` and `:X` forms, AND them with the tier filter and the terms, and keep the words in `{args}` for a command without `{filter}`: `TestRenderTierCommand`, the three new rows.
12. `filterArgument` drops the terms past the budget while the caller's filter still stands: **none** (F5). The existing budget row has no caller filter.
13. `takeCallerFilters` keeps a trailing `--filter` that has no value in the args: none. This is an edge case, and I filed no finding.
14. Planning refuses a caller `--filter` when the command spells its own `--filter`: `TestPlanTierRuns`, the exact-message row.
15. Planning's refusal for a hub without tier0 names `gate.tiers.tier0`: `TestPlanTierRuns`, exact row "a hub with no tier0".
16. `gateslot.Config.Tier` and `Config.Gate` refusals name the key: **none** (F3).
17. `holdsCallerFilter` and `filteredNoTestNotice` produce both wordings (other tiers declared, none declared) for `--filter` and `VSTestTestCaseFilter`: `TestRunTierNoTestNotice`, the two new rows. The `--filter=` and `--filter:` forms of `holdsCallerFilter` have no row, but this is minor and I filed no finding.
18. `withOtherTiers` fills `otherTiers` on both return paths of `planTierRuns`: **none** (F2).
19. The help text, the doc and the stencil prose are static prose and exempt.

**Findings**

- **F1 — MEDIUM, design.** Location: `internal/gatecli/tierrun_test.go`, `TestWithTimeoutMin`.
  - The fake clock's readings come from `time.Date`, so they carry no monotonic reading. For those, `at.Sub(start)` and `at.Round(0).Sub(start.Round(0))` are equal.
  - A mutant that drops the wall-clock term, `left := timeout - at.Sub(start)`, passes every row. That term is the core of the #215/#216 fix.
  - The stdlib cannot build two monotonic-bearing times whose wall and monotonic gaps differ. The test therefore needs a seam that gives the two elapsed counts separately, so a row can pin "monotonic 1m, wall 21h ends the context".
  - Evidence: `tierrun.go:477` (`max(at.Sub(start), at.Round(0).Sub(start.Round(0)))`) and `tierrun_test.go:634-650` (the readings built with `time.Date`).
- **F2 — MEDIUM, design.** Location: `internal/gatecli/tiers.go:834,843` (`withOtherTiers`).
  - No test asserts the `otherTiers` that planning computes. `TestRunTierNoTestNotice` injects `otherTiers` directly (`tierrun_test.go:580`), and grep finds no `otherTiers` in `tiers_test.go`.
  - A mutant that returns `runs` without `withOtherTiers`, on either return path, passes every test. In production the notice then always reads "the hub declares no tier besides those this call runs" and never names a tier.
  - Fix: add a `TestPlanTierRuns` row that asserts `otherTiers`, for example for a `--gate` key running tier0 with tier1 and tier3 declared, plus one row with a covered tier and one on the guard return path.
- **F3 — LOW, design.** Location: `internal/gateslot/tiers.go:388,415`.
  - The `Config.Tier` and `Config.Gate` refusals now name `gate.tiers.&lt;N&gt;`. Their rows assert only the substring "the hub declares no tier0" (`gateslot/config_test.go:883,918`), so the added key text is unpinned.
  - The mutation report's survivor at `tiers.go:415` confirms it.
  - Fix: make those rows assert the full message.
- **F4 — LOW, design.** Location: `internal/gatecli/counters.go:601` (`observeFailure`).
  - The new `vstestTotalTests` close of a failed test's block has no row: grep finds no `TestCounters` row with a failure block followed by `Total tests:`.
  - A mutant that removes the clause passes, and the summary lines would then be folded into the last failure's text.
  - Fix: add a `TestCounters` row whose failure block runs straight into a `Total tests:` summary, asserting the failure's text and the counts.
- **F5 — LOW, design.** Location: `internal/gatecli/tiers.go:895-907` (`filterArgument`, budget path).
  - When the terms are dropped past `greenCommandBudget`, the caller's `--filter` must still stand. No row combines a caller filter with terms past the budget, so a mutant that falls back to `tier.Filter` alone passes.
  - Fix: add a `TestRenderTierCommand` row with args `--filter Name=X`, a tier filter, and terms past the budget, wanting `--filter '(Category!=Slow)&amp;(Name=X)'`.
- **F6 — LOW, consistency.** Location: `internal/gatecli/tierrun_test.go`, `TestWithTimeoutMin`.
  - The redundancy report lists it as contained by `TestRunBuildTimeout`. Its assertions, the poll durations and the cause on a fake clock, are ones `TestRunBuildTimeout` never checks, so it pins what coverage cannot see.
  - Fix: mark it `//testtiming:keep the poll lengths and the cause of the timeout_min bound on a fake clock` directly above its `func Test…` line.

Tool calls:
1. One grep of the gateslot and gatecli tests for the tier0 refusal text, for `otherTiers` and for `Total tests`. My context held the diff's tests but not the existing assertions on those three behaviours.
