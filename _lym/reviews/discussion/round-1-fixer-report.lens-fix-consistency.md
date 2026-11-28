```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, criteria and cross-references in the decision record.

1. **LOW.** Changed text: D7's new reinterpretation paragraph. Stale twin: the Constraints line "`PATTERN-config-layers`: no config key moves; the new profile blocks are template additions the hub reconcile carries."
   - Summary: D7 now gives an operator-added profile its jacquard block through a one-time reinterpretation, not through the reconcile, but the Constraints line still credits every new block to the reconcile.
   - Evidence: D7 says "a profile the operator added that holds a tapestry block gains a jacquard block made from that tapestry block minus `loom.discussion`" through a reinterpretation registered in `internal/configreg`. The Constraints line was left unchanged.

2. **LOW.** Changed text: D5's new paragraph on the done-motif-run clear. Stale twin: D5's Bound, "the check reads the board once at start and nothing else."
   - Summary: D5 now has batten's start also read the slug's done motif run (the child recipe) to clear it, so "reads the board once at start and nothing else" no longer covers everything the start judgement reads.
   - Evidence: `startClearsMotifRun` calls `shedrun.ReadChildRecipe(location, task.Slug)` (`internal/battencli/startguard.go`). D5 now routes a needing recipe through that clear, and the Bound was left as it was.

Further tool calls: none; my context already held the changed record and the code it cites.
