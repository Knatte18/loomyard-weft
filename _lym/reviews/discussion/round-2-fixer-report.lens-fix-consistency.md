```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the change set against its sibling decisions, criteria and cross-references.

1. **MEDIUM.** D4's sentence "`lym loom decision add` before Record-Write has run refuses with its existing missing-record refusal" was left as it was, though the fix rewrote the three darn refusals right after it.
   - Evidence: `decisionWayNoRecord` (`internal/loomcli/decision.go:101`) still reads "on a darn run the record exists once Darn-Write has validated it, and until then message the running Darn-Write session with the call or, with no writer live, record the call as a board note…".
   - Under D4 the record exists once Record-Write writes it, so that text is stale.
   - Under the new D6 text, "a `Decision for &lt;slug&gt;:` note does not settle a stop", so its board-note route is stale too.

2. **MEDIUM.** The new D6 text says a no-parent stop is not settled by a note and that "the darn writer stencil drops its route of writing a note's answer into the record". It does not change the way forward that the stop report itself names.
   - Evidence: the stencil's no-parent bullet (`contracts/stencils/darn/darn-template-write.md:48`) has the stop report name "a `Decision for {{.slug}}:` board note that answers it and then `lym loom resume`".
   - D6 makes that change description the halt's reason. The halt would then carry a way forward that now loops.
   - The review's F4 fix asked for the stop report to name `lym loom decision add` then `lym loom resume`, and the fix left that part out.

3. **MEDIUM.** The new way forward on the first two darn refusals, "name `lym loom goto --to Record-Write` then `lym loom step`", disagrees with D4's sibling in-flight bullet.
   - The bullet's route is `goto --to Record-Write`, `step`, `goto --to Darn-Write`, `resume`, "a route that never passes Drift-Baseline after the writer committed code".
   - The refusal's route leaves the run's current row past Record-Write, at Drift-Baseline, with no step back to the row it was on.
   - A later `resume` then passes Drift-Baseline after the writer may have committed. The new D4 acceptance row repeats the same two-step route.

4. **MEDIUM.** The rewritten no-commit bullet now says the earlier pass's scope "stand[s] since … the earlier scope agent read the board entry through its drift subject". That disagrees with the new F9 sentence in the Drift-Baseline bullet above it.
   - The F9 sentence covers a run resumed without the goto: its Drift-Baseline agent is "pointed at a record that does not exist" and "may record a scope from the board entry alone, which then stands".
   - In that path the scope that stands came from an agent with no record, through the `decision-record` subject and not the board-entry subject.
   - So the no-commit bullet's justification is false for one of the two ways a run reaches Darn-Write with no record.

5. **LOW.** In D13 the new "Bound:" sentence sits mid-decision, between the `RetiredKeys` prune sentence and the `ValidateLocal` check paragraph, ahead of "the check below".
   - Every other decision puts `Bound:` after its `Rationale:` (D1, D4, D6, D10).
   - The unchanged sentence "refusing rather than ignoring keeps a run pinned to a driver from silently running another" still reads as unbounded, a few lines before the Bound says base-file values are silently dropped.

Calls made:
- grep for `decisionWayNoRecord` in `internal/loomcli/decision.go`: the context showed only the darn-writer and entry-lost constants, not the missing-record way forward that D4's unchanged sentence relies on.
