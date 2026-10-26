```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code and tests the fix changed (fix-code-correctness lens).

**Walked through:**
- `arbiterJudgesFirst` (`drivewiring.go`): the assignment `arbiterSeam.JudgesFirst = arbiterJudgesFirst` matches the field type `func(producer string) bool`, so the one caller still holds.
- `TestArbiterJudgesFirst`: with the defect (darn, finalize or any other row also marked) the false rows fail. With no row marked, the Webster row fails. The empty-producer subtest name gets `#00`, which is fine.
- New `TestRun_MasterSpawn` row: input is a record holding one `### Added after Discussion` entry. `DeclaredPatternChanges` finds no PATTERN section and returns nil. `AddedDecisionTexts` returns the entry, and `Run` passes it to `RenderMasterPrompt`. If the pass-through were dropped, the prompt would render `none` and the `Contains` check on the entry text would fail. The row therefore catches the defect.
- Reworked `master renders none` row: the `contains` string `"\n\nnone\n\n## A plan-drift"` is anchored to the next heading, so an earlier `none` cannot satisfy it. If the marker were filled empty it would render `\n\n\n\n## A plan-drift` and fail. An unfilled marker is caught by `notContains`. The earlier `inOrder` attempt matched `\nnone\n` at an earlier occurrence and failed, and that was fixed.

**Findings:**
- **LOW**: the second row F2 asked for, an unreadable record reaching `Run`'s new refusal in `internal/websterengine/runlevel.go` (the `AddedDecisionTexts` error branch), is not covered, and the fixer must say why in the report.
  - Evidence: `DeclaredPatternChanges` reads the same `DecisionRecordPath` first (`patternchanges.go:311-316`) and returns every non-not-exist read error with a re-entry way forward.
  - A directory path therefore stops at the earlier refusal and never reaches the new branch.
  - That branch is reachable only through a race between the two reads.
  - The disposition belongs in the fixer-report as fixed with an own change (pass-through row only) or deferred with this evidence, not as a silent skip.

No other findings.

Further tool calls: none; my context already held every changed file, the test output and the callee sources.
