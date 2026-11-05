```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 changes agree with the sibling decisions, criteria and cross-references they touch, and look for twins left unchanged.

**MEDIUM: the `command`-mode sentences in D2 and D8 leave out the records check**
- Changed text: D6 now says "Drift-Verify, Drift-Recheck, the drift gate entry and `lym drift check` first run the records check". That holds in both diff modes. D8's `lines`-mode sentence was updated to "runs D7's capture, records and seal checks … reports any records or seal failure".
- The problem: the `command`-mode twins were not updated. They still list only the baseline check, the seal check and the compare.
  - D2: "The writer's gate entry and `lym drift check` (D8) run the baseline check (D7), the seal check (D6) and the compare, report any baseline or seal failure, and pass".
  - D8: "In `command` mode it runs the baseline check (D7), the seal check (D6) and the compare, reports any baseline or seal failure, and passes".
- Why it matters: in `command` mode these two sentences contradict D6. Any stored `command`-mode rulings in `judged.json` (D2) are among the records D6 guards.

**LOW: D13's new source has no fallback for a run in flight**
- Changed text: D13 now reads "from the copies of `scope.json` and `judged.json` in the newest `Drift-Records` commit (D6)".
- The problem: D6's in-flight rule says "A run with no commit carrying `Drift-Records`, seeded before this lands, reads the files as they are until its first records write makes one". The Open risks line says "One already past Drift-Recheck lands with a verdict rendered from its old compare, acceptance and waiver fields (D13)". Such a run has no `Drift-Records` commit, so D13 as written has nothing to render from. It should take D6's fallback or point to it.

**LOW: the Open risks list of in-flight rules lacks the new records rule**
- Changed text: D6's new in-flight sentence, quoted in the finding above.
- The problem: the "A run past Drift-Expect when this lands …" item in Open risks lists each decision's in-flight behaviour, such as D7's "has no baseline digest and is checked without that guard". It has no matching line saying that such a run is not guarded by the records check until its first records write.

I made no further tool calls; my context held every text these findings cite.
