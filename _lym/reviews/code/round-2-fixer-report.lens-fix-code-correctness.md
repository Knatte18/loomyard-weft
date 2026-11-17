```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix-changed code (F1 `Select` loop and test row, F2/F3 fixture changes) with concrete inputs.

No findings.

- **F1 fix:** with runs a and b both landed at 10:00 and `MaxRuns` 1, `runs[1]` gives `firstLeftOut` = 10:00 and `cut` falls from 1 to 0.
  The widening loop now stops at `cut == 2` because of the `cut < len(runs)` bound, so there is no panic.
  The plan is slug `retro-20261116-100000`, `From` 07:00 (the `last` stamp) and `Until` 10:00, with runs a and b. That matches the new row.
- **F1 other input:** a, b and c at 10:00 with d at 11:00 and cap 2 widens to 3 and stops at d, so the existing row still holds.
- **F1 test against the old code:** the new row would hit `runs[2]` and panic on the old unbounded loop, so it fails with the defect in place.
- **F3 outliers:**
  - Turns are 6, 2 and 1, so the median is 2. Costs are 0.6, 0.5 and 1.2, so the median is 0.6 (the sorted middle element, exact).
  - s1 is an outlier on turns alone (6 ≥ 4, cost 0.6 < 1.2). s3 is an outlier on cost alone (1.2 ≥ 2×0.6, with equality exact in floating point, since doubling is exact; turns 1 < 4). s2 is neither.
  - With `and` in place of `or`, both outliers drop and the cell reads `-`. With a factor of 3, s3 drops (1.2 < 1.8), so the line changes.
  - The asserted cells follow from this: Runs cost `$2.3` (sum 2.3, rounded by `money`), median cost `$0.6`, and the producers line in session order, s1 then s3. The `usage.jsonl` and `sessions.jsonl` expectations match the new costs.
- **F2:** `lymdirs` is imported and used, no stale `_lym` literals remain in the edited calls, and the tier1 `tools/retro` test passed.
- **Callers:** the `Select` signature is unchanged, and the tier0 `retroengine` gate passed.

Tool calls: none; my context already held every changed file.
