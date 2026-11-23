```yaml
lens: leanness
outcome: reported
```

I reviewed the decision record and the support log for leanness, as the leanness lens. I found nothing BLOCKING: two MEDIUM findings and four LOW.

1. **MEDIUM — Constraints, third "Fact" bullet (commits d730f5544 and 3d3b48127).** This is history about how earlier key moves ran. The plan writer needs none of it to act.
   - Evidence: the requirement it serves is already in D5, which uses a `configsync` move table, and in the D5 and D6 rationales. The same history is in the support log under Turn 2 and Rejected alternatives.
   - Completeness-before-leanness: no constraint is lost if it moves. Cut it from the record; the support log already holds it.

2. **MEDIUM — Auto-mode assumptions repeat Decisions and Scope.** Five of the six bullets say again what D1, D4 and Scope "Out" already state, each marked with its decision id.
   - Evidence: the `pr` area, `<area>.model` and the `review` nesting are D1's table. "Refused rather than alias" is D4. The `run_settings` shape is in Scope Out, which says it a second time in the same record. The support log's question ledger Q1–Q8 says it a third time.
   - The spec requires the section, so keep it, but shorten each bullet to "<pick> (D<n>)" with no restated reasoning. Drop the `run_settings` line, since Scope already says it.
   - The bullet that reads "by a reinterpretation" as "automatically, on the reconcile" is the one assumption that carries new information, a reading of the entry's own words. Keep it in full.

3. **LOW — D2 rationale lists other templates' prefix groups** (`gateslot`'s `redundancy_*`, `landingshed`'s `finalize_fix_*`, shed's `tackler_*`). That list is the deliberation behind rejecting them. The support log's Rejected alternatives already carries it.
   - Cut D2's rationale to the operator's-example reason and "no other file shares loom's perch machinery".

4. **LOW — D4 rationale argues against a rejected alternative.** "a silent warn-and-ignore of an old key would drop a tuned value with no signal" restates the support log's rejected "Warn and ignore an old key". The compaction rule keeps rejected alternatives out of Decisions.
   - Rephrase it as a positive reason, for example: a tuned value is never lost without a signal, and `PATTERN-refusal-way-forward` wants a refusal with a way forward.

5. **LOW — Acceptance criterion "D2: every other module's template keeps its key set unchanged".** This criterion pins an absence of change and has no behaviour to exercise. Any test for it would restate the templates.
   - Better: a `Review:` bullet, or cover D2 by naming it in the D1 template-check bullet.

6. **LOW — Constraints, first Fact bullet: "its one caller reads `beam.yaml`".** The claim is true today: the only production caller is `internal/dobbyengine/tasksetting.go`, and it reads beam keys.
   - It is a perishable enumeration of callers, against the prose rule. Rephrase it as "D2 leaves no nested key that `configengine.TopLevelValue`'s readers read".
   - The same bullet's "D4 to D7 extend the pattern as `## PATTERN changes` states" points to another section without adding anything. Cut it.

Kept on purpose: D1's mapping table is the design itself. Both Open risks are for the parent to settle. The two plan-writer notes are actionable hints.

Tool call: one grep for `TopLevelValue(` callers outside tests, to check the "one caller" claim; my context held only the two subagents' differing summaries.
