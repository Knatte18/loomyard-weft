```yaml
lens: leanness
outcome: reported
```

Leanness lens review of the decision record and the support log, looking for content to move or cut.

- **MEDIUM — D1 Rationale, D11 Rationale: rejected alternatives sit inside Decisions.**
  - Evidence: D1's second rationale sentence ("A key move would make every tapestry run read jacquard keys and needs a move table…") argues against an option that was not chosen. The support log's Rejected alternatives already carries it ("Moving `tapestry.*` keys to `jacquard.*`…"). D11's "Written before landing, it would name a recipe the hub cannot run yet" is the same, and is repeated as "A card writing the nickname… writes Bolt before the recipe lands".
  - The spec's compaction rule says Decisions carry only the decision and its rationale, never a rejected alternative.
  - Fix: cut both sentences from the record. The support log keeps them, and no requirement is lost.
- **MEDIUM — Open risks, second bullet: this is settled behaviour, not a risk for the parent.**
  - Evidence: "A tapestry run parked at Preflight across the upgrade… needs the goto D4 names" restates the last paragraph of D2 and D4's way-forward clause. Open risks is the channel for questions only the parent can settle, and this one has no question.
  - Fix: cut the bullet. D2 and D4 already state the requirement.
- **LOW — Scope "In" list mirrors D1–D10 one for one.**
  - Evidence: each In bullet restates a decision's subject: jacquard file (D1), tapestry rewrite (D2), Expand (D3), stored-only (D4), refusal (D5), raddle (D6), profiles (D7), lists (D8), cost/glossary/docs (D9, D10).
  - This breaks "say it once". Scope could name the boundary in a sentence or two and leave the items to the Decisions.
  - The Out list carries real exclusions and stays.
- **LOW — Goal repeats D1's row enumeration.**
  - Evidence: the Goal lists "Preflight, the record seed, Drift, Plan, … Friction-Reflect", and D1 lists the same rows.
  - Fix: write "tapestry's rows after Discussion" in the Goal and keep the list in D1 only.
- **LOW — Auto-mode assumptions restate the decisions they come from.**
  - Evidence: all six bullets duplicate statements already in D1, D2, D6, Scope Out, D7 and D11.
  - The section is required, but each bullet could be a one-line pointer naming its decision id, without restating the content.
- **LOW — Constraints "Fact" bullets repeat D3 and D4.**
  - Evidence: the Expand fact ("an outermost block row with an empty `on_done` is refused today, hence D3") restates D3's premise. The Record-Write fact restates D4's rationale.
  - Keeping them as the evidence pin is defensible, since the rubric's unpinned-fact rule asks for one. Then D3 and D4 should drop their own restatement.
- **LOW — D6 last line and Scope Out both say the `tapestry` shape keeps its name.** Keep it in one place.
- **LOW — the second plan-writer note duplicates `relevant.md`.**
  - Evidence: the `startguard.go` hint repeats the relevant list's entry for `readRaddleStartRefusal` and `raddleStartRefusal`. Cut it.
- **LOW, emphasis — D10's glossary line adds "Never: build recipe", which no one asked for.**
  - Evidence: the entry does not ask for a ruled-out word, and the Sketch glossary line shows that "Never" is optional.
  - Fix: drop it unless "build recipe" was in real use.
- **LOW, emphasis — D5 carries an unneeded sentence.**
  - Evidence: "the refusal text never names a recipe the check did not read" is an awkward restatement. The slug and recipe are already filled from the entry and the constant, as the Constraints `PATTERN-code-conventions` bullet says. Cut it.

Further tool calls: none.
