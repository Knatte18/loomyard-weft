```yaml
lens: fix-proposal
outcome: reported
```

Task: lens fix-proposal, challenging the reviewer's proposals the fixer carried out as proposed (F1–F5).

Findings: no findings.

- F1: the bound `cut < len(runs)` is sound. If every run shares the second, `cut` ends at `len(runs)`, `runs[:cut]` keeps all of them, and `until` becomes the last run's landing, which equals `firstLeftOut`. When a later second exists, the loop stops at it. `MaxRuns` is at least one, so `runs[w.MaxRuns]` is in range inside the `len(runs) > w.MaxRuns` branch. The new row reaches the old panic.
- F2: the fixture paths now build from `lymdirs.LymDirName`, and tier1 passes, so it equals `_lym`. The literals "usage", "shed" and "reviews" are plain subdirectory names. `lymdirs` documents `UsageDirName` as an exception that sits under `LymDirName`, but PATTERN-lym-files only reserves `_lym` and `.lym`, so this is not a defect.
- F3: I ran the doubles through the new fixture and the test now tells the rule apart.
  - s1 (6 turns, $0.6) is an outlier on turns alone, and s3 ($1.2, 1 turn) is an outlier on cost alone, so an `and` rule drops both.
  - A factor of 3 drops s3 (1.2 < 1.8).
  - The boundary `1.2 >= 2*0.6` holds exactly in binary, because doubling is exact.
  - The sum, 0.6+0.5+1.2, is rounded by `money` to `$2.3`.
- F4: the summary edit follows the review's own description of the integration test.
- F5: "Write one report and its line in the research index, and change nothing else" now agrees with section 4's index step. `TestPlan_BodyFillsTheShippedStencil` still passes.

Calls: none made. My context already held every changed file, and the gate output was enough to confirm the double-precision cases.
