# Support log — card-own-entry-edit

## Interview

Turn 1 (autonomous, no advisors): read the board entry, its operator decision and the board list; no other entry touches card planning or the board lock.
`stencil-intent-audit` audits stencil claims in general and does not cover this rule.

Turn 2: traced the creel-core plan in `loomyard-weft`.
Card `12-hubgeom-creel-geometry.md` in its first plan generation carried no board step itself; the creel-core decision record's doc decision said the `creel-core` board body is cut to a pointer at `internal/creelengine/doc.go` in the same task, citing `CLAUDE.md`'s docs-in-the-same-commit rule and `PATTERN-documentation-lifecycle`.
The `CLAUDE.md` sentence "a design whose code has landed moves from the board entry into `doc.go`" and the matching `docs/overview.md` bullet are the source.
The Plan stencil reads only the decision record and never the board, so the step entered at Discussion and was carried into a card.
`contracts/specs/loom-plan-spec.md` and `loom-discussion-spec.md` hold no wording about editing a board entry or moving a design out of one.

Turn 3: settled the decisions as auto-picks (see ledger).
Decision numbers below Turn 3 are those of the record at the time; round 2 dropped the PATTERN decision, so the Plan, darn and rubric decisions are now D3, D4 and D5.

Turn 4: lens review by forks; every fork returned a report.

- premises, MEDIUM: D2 claimed the advisor stencil includes the discussion stencil; only the chair does. Fixed: the record names the advisor as left unchanged on purpose.
- premises, LOW: the landing and prune premise holds; the contracts tests exist. No change.
- board-fit, LOW: no entry duplicates or contradicts the record; `creel-agents` may overlap at the line level on the discussion stencil. Rejected as a dependency: the overlap is a merge, not a design conflict.
- goal-scope, MEDIUM: the darn rubric was missing from the rubric decision. Fixed: added to D6.
- goal-scope, LOW: the specs were not reported as checked. Fixed: Turn 2 states them clean.
- goal-scope, LOW: the check rejection was thinly argued. Fixed: the Scope Out line gives the slug-match miss.
- goal-scope / scenarios / pattern-fit, MEDIUM (merged): the Plan writer dropping a decided step on its own was an unbounded routing edge and extended the plan format. Fixed: D4 now uses the stencil's existing parent route, and the Plan-review line accepts that drop.
- goal-scope, LOW: darn and rubrics go past the entry's letter. Rejected: same root cause, kept as an auto-pick.
- scenarios, LOW: no criterion pinned that other entries and board code stay allowed. Fixed: criterion added.
- scenarios, LOW: rework and prior-plan paths. Fixed first by a PATTERN entry; since round 2, by D1's `CLAUDE.md` rule, which every Plan writer, darn writer and rework session loads.
- scenarios, LOW: in-flight runs keep old seeded stencils. Fixed: now the open risk.
- leanness, MEDIUM: the no-check decision appeared three times and was a rejected alternative written as a decision. Fixed: one Scope Out line, decision and auto-mode bullet cut.
- leanness, LOW: D1 rationale retold history, the Goal repeated Scope, the open risk restated a decision, and a constraint repeated a criterion. Fixed: all four trimmed or cut.
- leanness, LOW: the darn auto-mode bullet repeats D5. Rejected: the auto-pick flag stays visible to the plan writer.
- pattern-fit, MEDIUM: D1 rewrites the text `PATTERN-documentation-lifecycle` points at. Fixed: D1 names the amendment.
- pattern-fit, MEDIUM: a new cross-cutting rule needs a PATTERN entry. Fixed then by a new D2; round 2 dropped it and moved the rule into D1's `CLAUDE.md` text (see Rejected alternatives).
- pattern-fit, LOW: the overview heading anchor is linked elsewhere. Fixed: constraint added.

## Rejected alternatives

- Let the holding run edit its own entry's body through `lyx board upsert`: the operator decided the lock stays.
- A `validate-plan` check refusing a card that names `lyx board`: false refusals on board-module tasks; a slug match in free text is no better.
- Rewording `PATTERN-documentation-lifecycle`'s own line: it says only where a built design lives and is not what misled the writer; D1's `CLAUDE.md` text states the new rule.
- A new `PATTERN.md` entry for the rule: `internal/pattern/doc.go` says PATTERN text states rules for the code only and never how an agent, a run, a stencil or a workflow uses it, and this rule governs decisions, cards and steps with no code anchor.
- Fixing only the Plan stencil: the step was born in the decision record, which the Plan writer must follow.
- Letting the Plan writer drop an own-entry step on its own and note it in the overview: an unbounded departure from the record and an addition to the plan format.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where did the step come from? Resolved: creel-core's decision record, citing `CLAUDE.md`'s doc-lifecycle sentence.
- Q2 Which texts change? Auto-pick: `CLAUDE.md`, `docs/overview.md`, loom Discussion and Plan stencils, darn write stencil, and the Discussion-review, Plan-review and darn-review rubrics. Rationale: each is a place a writer or reviewer reads the rule.
- Q3 Include darn? Auto-pick: yes, same defect with no card in between.
- Q4 Mechanical plan check? Auto-pick: no, a free-text match gives false refusals and misses command-free wording.
- Q5 Change the PATTERN entry? Auto-pick: leave `PATTERN-documentation-lifecycle`'s line as is and add no PATTERN entry; `CLAUDE.md` carries the rule.
- Q6 How does Plan treat a decision that asks for an own-entry edit? Auto-pick: ask the parent to record the drop, as for any post-Discussion scope change.
