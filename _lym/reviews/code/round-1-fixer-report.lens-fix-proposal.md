```yaml
lens: fix-proposal
outcome: reported
```

**Lens fix-proposal: findings**

1. **LOW** — `docs/research/speed/coverage-map-pilot-2026-11-12/report.md:42`, the F7 clause "one more than the 34 map entries".
   - The review asked for a clause saying why `tests` is 35 against 34 entries.
   - The fixer's clause only restates the difference. A reader still does not know why the verb counts 35, so the premise stays open.
   - The report states 34 mapped tests (line 23) and 34 entries (lines 37 and 116) throughout. The new clause leaves 35 unexplained.
   - The sound change is either to give the cause or to drop the `tests` figure, the review's other option.

2. **LOW** — `internal/gatecli/testmap.go:375` is the twin of the F6 rewording.
   - It still reads "set the dir of the lowest of tier0 and tier1 to the Go module's root".
   - `redundant.go` now reads "declare tier0 or tier1 with its dir at the Go module's root".
   - That text sits in the same `gatecli` package, so the two verbs now word the same way forward differently.
   - `testmap.go:375` is not covered by the F6 finding, so changing it is optional. The inconsistency is now visible between the sibling verbs.

3. **LOW** — `internal/testcover/verdict.go:121-123`, the F4 doc comment.
   - The new line "and counts only judged entries as covers" duplicates the keep and outcome sentences that follow.
   - Those sentences already say that keep-marked and outcome entries are "neither judged nor counted as a cover".
   - This is padding, not a wrong statement.

The other proposals hold:
- **F1:** the two new refusal rows passed in the tier1 run, so each reaches its own branch.
- **F2:** the key is omitted when `Failure` is nil, and the extended test pins it.
- **F5:** `Tier: 1` on `TestCovSub` now exercises the tier mapping.
- **F3:** I found no problem with the new index line.

Tool calls (2 commands, one grep and one pattern search):
1. `grep` for the old F6 wording across the `.go` and `.md` files: my context lacked any check for a twin of the reworded text, and it found `testmap.go:375`.
2. `grep` in the pilot report for "35", "34" and "verdict": my context held the F7 row only, not the report's other uses of the 34 entry count.
