```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of `decision-record.md` and `support-log.md`, looking for content to move or cut.

**MEDIUM: D2, D3 and D4 argue against rejected alternatives inside `## Decisions`.**
- **Evidence:** D2's rationale says "A row per batch cannot exist in a fixed recipe, and shed's `Yield` ends the driver's loop". D3 says "a new DAG batchifier belongs to the parallel step". D4 ends "without a separate pack". The spec's compaction rule says Decisions carry Decision + Rationale only, and rejected alternatives go to the support log.
- **Already in the support log:** all three appear under `## Rejected alternatives`. They are deliberation, not a constraint the plan writer needs, so the completeness test lets them go.
- **Fix:** cut these clauses from D2, D3 and D4. Each rationale keeps its positive half: the operator's 2026-11-15 decision, the board's "batches formed as the run goes", and the recovery-strand shape.

**LOW: Goal restates D2, D5 and D8.**
- **Evidence:** the Goal paragraph names the ladder, the stop chain (rule table, tackler, arbiter, parent) and the journal's purpose. D2, D5 and D8 each state these again.
- **Fix:** cut the Goal to what Weaver is, a cold agent per batch with ladder escalation and no master, and leave the mechanisms to the decisions.

**LOW: Scope "In" and "Out" repeat decisions.**
- **Evidence:** "In" has "Lifting the Webster batch machinery…", which is D11, and "`implementer-spec.md` gains the `weaver` row", which is D1 plus the Review criterion. "Out" has "A kick-start pack: the batch prompt names the few files…", which is D4.
- **Fix:** keep the Scope bullets as one-line boundary markers and drop the explanatory tails.

**LOW: D2's list of Stuck causes repeats D5, D6 and D10.**
- **Evidence:** D2 lists each Stuck trigger, citing D5, D6 and D10. D6's last bullet, the plan-conflict reroute, also repeats D1's routes.
- **Fix:** have D2 say the row returns Stuck "only on the stops D5, D6 and D10 name, or a fault in Weaver's own config or state". Cut the plan-conflict line from D6, or keep it in one place.

**LOW: `## Auto-mode assumptions` repeats each decision verbatim.**
- **Evidence:** each bullet repeats a decision's text with its D id.
- **Fix:** this is acceptable as a pointer list. Shorten each bullet to "the pick" plus its D id, which most already are. No action is strictly needed.

**LOW: two Constraints entries are generic.**
- **Evidence:** "PATTERN-code-conventions: Weaver's package doc documents only Weaver; config names are named constants" and "PATTERN-sole-parsers: the plan is read only through `internal/planparser`". D3 already says the plan is read through planparser. These bind every task alike and add no Weaver-specific bound.
- **Fix:** cut the sole-parsers line, since D3 covers it. The code-conventions line can go too.

**LOW: Notes for the plan writer overlaps `relevant.md`.**
- **Evidence:** the second note, `internal/loomshed/webster.go`, repeats the relevant list's entry for that file and why it matters.
- **Fix:** drop the second note. The first note adds members the relevant list lacks, so it can stay or move into `relevant.md` as members of a websterengine entry.

**Other (emphasis only): an open risk the writer can settle.** The third Open risk, about the journal push belonging to `push-before-act-takeover`, is not a parent question. D9 already states the bound ("whenever the run's records are pushed"), so the line could become a Constraint. It is a `consistency`-style nit, not a leanness one.

I made no further tool calls.
