```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that should move or be cut.

- **LOW, Notes for the plan writer, third bullet.** "Gate findings files are kept per row by `KeepGateFindings`; the writer seats' gate findings need a path per seat." This is plan-level wiring detail that Plan computes fresh. It is not a design decision, so cut it or fold it into D6 as a one-clause requirement ("each writer seat's gate findings are kept at a path of its own").
- **LOW, Notes for the plan writer, second bullet.** "the prior-plan amendment goes to the chair" restates D6, which already says the fresh-spawn preparation runs `loomshed.NewPlanDirRotator` "as `Plan-Write`'s does". Cut it, or make it a single clause in D6.
- **LOW, Open risks, third bullet.** "A crash mid-step restarts every part writer…" repeats D6's stated bound ("a crash that loses any writer's strand ends the step on resume, and the rerun starts every part over"). Keep one copy; the bound in D6 is the one the plan writer needs.
- **LOW, D5 rationale, last sentence.** "`Required` has no production user, so its line rule changes for writers without breaking a caller" is exploration evidence that belongs in the support log. However, it also carries a constraint: the change to the required-seat rule may not alter any existing caller. Keep it in the record as a Constraints line ("no production table sets `Required`; advisor tables keep today's notice rules") and move the "why" to the support log, where Turn 1 already holds it.
- **LOW, D9 rationale, second clause.** "making the plan overview its output would mark the chair done the moment the join wrote it" is a rejected-alternative argument. The support log already records it as a rejected alternative and as Q10. Under the completeness test the requirement stays: the chair's output is `join.md`, written last. Cut the rest.
- **LOW, D10 and D11.** The two stencil decisions are bullet-for-bullet stencil content. The board asks Discussion to settle that content, so it belongs in the record and is not a relocation. D11's "write its `00-overview.md` last and run `validate-plan --part`" repeats D7's self-check statement, which is acceptable because it is stencil wording. No finding.
- **LOW, Auto-mode assumptions.** "Opus medium default", "parts_max 6" and "no twill forks" each repeat D2 verbatim. That section should hold only assumptions made because no operator answered, and the support log's ledger already marks them as auto-picks. Shorten each to a pointer, such as "D2's model defaults and `parts_max`, D4's one-part fallback: auto-picked, see ledger", or keep one line per pick without restating the values.
- **MEDIUM, Scope, In list.** This list mostly restates the Decisions one by one ("Two recipe rows…", "New tapestry config keys…", "The parts file…"), so it is the same content twice. The Out list carries real exclusions. Trim In to one line ("everything D1 to D13 decide") or to the items no decision names. As written, every list item must be kept in sync with the decisions.
- **LOW, Goal.** The second paragraph re-describes D1, D8 and D9. That is acceptable framing, but the clause "which the existing `plan` gate, Plan-Review and Webster take unchanged" also appears as a constraint and in D9. Keep it in one place.

Additional calls made: none.
