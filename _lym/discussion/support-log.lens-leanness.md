```yaml
lens: leanness
outcome: reported
```

Task: leanness review of the decision record and support log, looking for content to move or cut.

1. **MEDIUM** — D4 Rationale. It argues against the rejected option ("records on a second branch … needs `_lym` excluded … `git add -A` stages it …"), and that option already appears in the support log's Rejected alternatives. Keep the two facts the plan writer needs (the operator's rule that `_lym` is never excluded, and that lym never force-adds) as one Constraint line. Then cut the narrative about the alternative from D4. Evidence: the compaction rule "Decisions carries Decision + Rationale only. Never list a rejected alternative here."

2. **LOW** — D9 last line, and the second Open risks bullet. Both say the same thing, that a missed reader costs a re-run or a false dirty refusal. Keep the bound in D9 and cut the risk bullet, or the reverse. Evidence: the prose rule "Say it once".

3. **LOW** — Auto-mode assumptions. Each bullet repeats a decision or a Scope "Out" line: D4, D1, D8, conversion out of scope, built now. Reduce each to a one-line pointer naming the decision id and the fact that it was an auto-pick, or drop the bullets that Scope already states (conversion, building now). Evidence: the section mirrors D1, D4 and D8 and Scope "Out" bullets 1 and 3.

4. **LOW** — Constraints. Several lines only restate that decisions hold: "locks, push order … hold in both layouts (D6, D7)", and `PATTERN-landing` "the pull-request resolver is unchanged". The plan writer gains nothing it can act on from them. Keep the lines that bind an implementation choice: `lymdirs` as sole declarer, the force-push through `gitrepo`, and the named constants. Trim the rest.

5. **LOW** — D3, the `ResolveWithin` sub-bullet ("has nothing to match"). It states no behaviour to build or test. Cut it, or say what `ResolveWithin` does in a one-repository hub.

6. **LOW** — D6, "Store housekeeping runs once, on the one store." This follows from the layout and carries no requirement. Cut it, unless it is meant to say that the second-store gc and config writes must be skipped without error. If so, say that.

7. **LOW** — Notes for the plan writer, third bullet (the `ls-tree`/`mktree` recipe). This is plan-level mechanics. It is allowed as a hint, but it adds no design input. Cut it or keep it, at the writer's discretion.

8. **LOW, consistency (other)** — Acceptance criteria, the Review bullet "D2, D12: docs…". D12 covers tests and fixtures, not docs, and Scope lists "Docs for setting up a one-repository hub" with no decision owning it. Either tie the docs to D2 alone, or add the doc requirement to D2's text so the criterion has a decision behind it.

9. **LOW (other)** — Notes for the plan writer, first bullet. It lists background-file edits for `PATTERN-fabric-boundary`, `PATTERN-lym-files`, `PATTERN-cwd-resolution` and `PATTERN-pair-remote`. That is a requirement for the Webster card, not a hint, and Notes are "never a completeness requirement". Move it to Constraints, or to a sentence under PATTERN changes, so it is not lost as optional.

I made no further tool calls; everything needed was already in context.
