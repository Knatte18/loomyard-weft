```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens on the `_lym/landing/summary.md` counts fix (F1).

1. **LOW: the Numbers table still shows pre-restoration timings and test counts.**
   - Changed text: the "After the last commit" column of the table in `_lym/landing/summary.md`: `44.2 s`, `34.2 s`, Passed `1366`.
   - The reviewer's fix only asked for the dropped and kept counts and the kept list. Applied as proposed, it leaves this column stale.
   - The column is headed "After the last commit". Three fix commits (`9e969b84`, `b8308134`, `4612393b`) restored about ten tests after the measurement, so Passed can no longer be 1366.
   - The "Design decisions" line "ran the tier 1 gate twice, only to take the before and after times" now describes a measurement that predates the final tree.
   - Suggested fix: rerun the gate once and update the column, or relabel it as measured at `feaa74d0`.

2. **LOW: the dropped-cost sentence was removed rather than recomputed.**
   - Changed text: "Their standalone cost in the study is no longer totalled here, since ten of the originally dropped tests came back."
   - The 51.5 s figure was recomputable from the study's `greedy.tsv` by subtracting the ten restored tests' costs, but the rewrite just drops it.
   - The following paragraph ("most dropped tests cost 0.2 to 0.5 s each standalone") still leans on that cost.
   - This is the fixer's own change, and the report should mark it as fixed differently from the proposal.
   - Suggested fix: recompute the total, or say plainly that it is unrecomputed.

3. **No finding: the 188 and 64 figures differ from the reviewer's 187 and 65, and the evidence supports the fixer.**
   - The reviewer's 65 counts `+` lines in `f2ac2bf7..HEAD`, which includes the reworded `TestCwdMutation_MigratedFilesStayChdirFree` "proves" line.
   - That line is not a restored test: the diff against `feaa74d0` shows it as a `-` and `+` pair.
   - The same diff shows ten genuinely new keep lines, so 54 + 10 = 64, and 198 − 10 = 188.
   - In scope stays at 252 (188 + 64), which is consistent.
   - The report must mark F1 as fixed differently from the proposal, naming the 187 and 65 figures not used and the reason.

Tool calls: none; my context already held everything this lens needed.
