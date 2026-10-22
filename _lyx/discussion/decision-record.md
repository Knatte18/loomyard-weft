# Decision record — research-in-discussion

## Goal

A factual question the design rests on is researched and answered in Discussion, with its evidence, and never handed to Plan or to an implementer with a halt-on-discovery rule.
The Discussion and Plan reviewers flag a design or plan that defers one.

## Scope

In scope, all stencil prose under `contracts/stencils/loom/`:

- The Discussion writer stencil, `loom-template-discussion.md`, which the seat-table chair stencil includes whole.
- The Discussion-Review rubric, `loom-rubric-discussion-review.md`.
- The Plan-Review rubric, `loom-rubric-plan-review.md`.

Out of scope: the Plan writer stencil, the advisor stencil, the darn stencils and rubric, every Go file and the discussion spec.

## Decisions

### D1 — The Discussion writer researches the facts the design rests on

The writer stencil's exploration step gains a rule: a factual question the design rests on — tool or dependency versions, upstream behaviour, what the code does today — is researched in Discussion, from the code, upstream changelogs, manuals and docs at the versions that matter, and its answer is recorded with the evidence it rests on.
The answer goes into the decision record's Decisions or Constraints with its evidence named (source and version), so the plan writer gets a settled fact.
The writer never leaves such a fact for Plan or an implementer, and never hands downstream a rule to halt when a later step discovers it.
A question research cannot settle, because it needs a choice rather than a fact, goes to the interview as before.
Rationale: the operator decision of 2026-10-19; a deferred fact found by an implementer costs a failed batch, a recovery and a re-plan, where the same research in Discussion costs minutes.
Bound: the rule widens the exploration step only to the facts the design rests on.
The step's existing exclusions stay: no exact signatures, `file:line` citations, interface shapes, dependency lists or exhaustive existing-pattern research.

### D2 — The Discussion reviewer flags a deferred research question

The Discussion-Review rubric's `## Also flag` gains a bullet: a Decision, Constraint, risk or acceptance criterion that leaves a fact the design rests on to be found out later, or a halt-on-discovery rule handed downstream, is a finding.
The fix is the research done and its answer recorded with its evidence.
A fact the writer stencil tells it not to gather is no such question, so the rubric's writer/reviewer symmetry note keeps holding.
Rationale: the writer rule alone has no check; the review is where a deferral is caught before Plan.

### D3 — The Plan reviewer flags a card carrying a research step

The Plan-Review rubric's `## Also flag` gains a bullet: a card that carries a research step whose answer the design depends on, or a rule to halt when its work discovers such a fact, is a finding.
The finding names the fact; its fix is that fact researched and recorded in the plan, or in the decision record through the parent when it changes the design.
Rationale: Plan is the last gate before an implementer would meet the question.

## Constraints

- Stencil prose follows `scribe:prose`: one sentence per line, no padding.
- No test asserts a stencil's static prose (`PATTERN-test-economy`); the existing rubric marker tests keep passing, so neither rubric gains a marker.
- The writer stencil gains no marker and no `{{if}}`/`{{range}}` conditional, as its leading comment requires.

## Auto-mode assumptions

- The rule's examples follow the board entry's list: versions, upstream behaviour and what the code does today.
- Evidence is recorded as a named source and version, never as an excerpt.

## Open risks

- The writer's Step 2 exclusions and the new research rule sit side by side; a writer could read "no exhaustive research" as licence to defer a fact.
  The new rule names its relation to those exclusions to close that reading.

## Acceptance criteria

- The Discussion writer stencil tells the writer to research a fact the design rests on and record its answer with evidence, and forbids deferring it or handing down a halt-on-discovery rule.
- The Discussion-Review rubric lists a deferred research question and a halt-on-discovery rule as findings.
- The Plan-Review rubric lists a card carrying a research step the design depends on, or a halt-on-discovery rule, as a finding.
- `lyx gate test` over the changed scope stays green, including `contracts/stencils`' rubric tests.
