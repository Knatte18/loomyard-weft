```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out, F1 and F2, through the fix-proposal lens.

**[MEDIUM] F1: the gate is told to count something it cannot see.**
- **Changed text:** the new D3 bullet, "so the count of failing results since the last pass counts shuttle arrivals, not inner gate runs."
- **Summary:** the proposal tells the gate to count shuttle arrivals, but the gate has no way to tell an arrival from the wrap's re-run of it.
- **Case:** a verify fails mechanically and the wrap spawns a fixer. `waitThenRun` then calls `inner()` again (`internal/gatefixer/rowgate.go:218`), exactly as `spawning` called it the first time (`rowgate.go:153`). Neither call passes an arrival marker or a counter. If the fixer's tree still fails, the gate sees two failing results from one arrival and cannot tell them apart. The record has no channel, such as an arrival number handed to the closure or a count the wrap keeps, through which the gate could count arrivals.
- **Evidence:** `internal/gatefixer/rowgate.go:144-166` and `rowgate.go:213-233`. `spawning` and `waitThenRun` both call `inner()` with no arguments.

**[LOW] F2: the Goal's build order puts the trial and D10 in this run's sequence.**
- **Changed text:** the Goal, "then the trial, then the `mutation` step (D10) only if the trial pays."
- **Summary:** Plan-Write reads only the record, and the Goal's build order now lists two steps that no implementer of this run builds.
- **Case:** a plan writer following the build order could plan a card for the trial, which needs a worktree per landed commit that no implementer may create (D11 rationale). It could also plan a conditional D10 card. The Goal does not say that the trial and D10 lie outside this run; only D10's opening sentence and the D11 board note say so.
- **Evidence:** the added entry's edit (1) and D11's rationale, "a worktree per landed commit, which no implementer of this run may create."

I checked the rest and found nothing wrong:
- **The F1 bypass with a fixer.** When a fixer's re-run passes the verify and then yields the reach turn, `waitThenRun` returns that result to the writer, so the bypass holds there too.
- **The F2 config-check removal.** A hub listing `mutation` is refused before D10 is built, because the config check accepts only `reach` and `redundancy`.

Further tool calls: none.
