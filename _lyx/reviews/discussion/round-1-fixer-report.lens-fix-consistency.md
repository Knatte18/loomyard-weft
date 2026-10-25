```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency check of the round-1 change set to `decision-record.md` and `support-log.md` against the text around it.

1. **MEDIUM.** D4's new sentence "Run by a caller at a `Drift-Await` halt, `lyx drift capture` runs in the caller's foreground" conflicts with two passages the fix left unchanged.
   - D4's opening says "The capture is a detached `lyx drift capture` child of the row, registered with `runproc.ChildRecord` under a new role `capture`".
   - The Constraints line says "`pairteardown` ends a pair's processes, the capture included, through `runproc.EndSparing`".
   - The record does not say whether a capture run by a caller is registered under role `capture`. If it is not, the claim that teardown and adopt end "the capture included" fails for this new route.
   - The D5 adopt paragraph says "the run's processes on the old machine are ended, a running capture included", which carries the same gap.
   - Either state that the caller's capture registers itself under role `capture`, or narrow both sentences.
2. **LOW.** D7 still says "so only that agent, the arbiter or the operator at that halt can accept or withdraw" with no qualifier.
   - The new D2 line 58, D6 line 156 and D7 line 171 say that under scope `full` the arbiter never receives a drift halt.
   - Read alone, that sentence still offers the arbiter as an accepter under `full`. A reader would notice, but D7 lines 170-171 correct it two lines later.
3. **LOW.** The support log's question ledger misses the new auto-picks.
   - Q11 and Q12 were updated in the ledger.
   - It has no entry for the new auto-pick `lyx drift withdraw`, which the Auto-mode assumptions now list. It also has none for the two route choices: the recapture through `lyx drift capture` plus resume at a `Drift-Await` halt (F4 option b), and the deferral of regression to a second halt at `Drift-Verify` (F8 option a).
   - The ledger records every other auto-pick, Q1 to Q12, so these three stand out.

Tool calls:
- `cat -n` of the decision record's lines 50-70 and 120-175, to read D2, D5, D6 and D7 as one changed whole. My context held only the edit fragments, not the paragraphs around them.
