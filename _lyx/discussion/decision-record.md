# Decision record — card-own-entry-edit

## Goal

No step of a task's run is asked to change that task's own board entry's body or scope, which the run's board lock refuses.
Throughout this record, editing the own entry means such a change; the landing's status write is outside it.
The entry's `run_settings` are outside it too, since the lock never compares them.
Scope means the other fields the run lock compares, such as title, kind, recipe, priority, isolated, brief, short name, labels, issues and dependencies.

## Scope

In:

- The documentation-lifecycle wording in `CLAUDE.md` and `docs/overview.md#documentation-lifecycle`.
- The loom Discussion stencil, the loom Plan stencil and the darn write stencil.
- The loom Discussion-review, Plan-review and darn-review rubrics.

Out:

- The board lock and `lyx board upsert`: unchanged, per the operator's 2026-10-15 decision.
- A mechanical plan check in `lyx loom validate-plan`: a card is free text, a card naming `lyx board` also fires on every board-module task, and the defect's own wording ("the board body is cut to a pointer") names no command.
- A `PATTERN.md` entry: PATTERN text states rules for the code only, and this rule binds agent artifacts.
- Board writes to other entries, such as filing a follow-up entry, and changes to the board module's code; they stay allowed.

## Decisions

### D1: The doc rule says the task's own entry is left alone

**Decision:** `CLAUDE.md`'s "Docs land in the same commit" rule and the `docs/overview.md` documentation-lifecycle bullet say that the task building a design writes it into the package's `doc.go` and never changes its own board entry's body or scope: the landing marks the entry done and the board prune removes it, so no pointer is left behind.
The `CLAUDE.md` rule also states that no decision, card or step of a run changes the running task's own board entry's body or scope, because the run's board lock refuses such a change.
This amends the full statement that `PATTERN-documentation-lifecycle` points at; the entry's own one-line text and its background file stay as they are, since they say only where a built design lives.
**Rationale:** the current sentence "a design whose code has landed moves from the board entry into `doc.go`" reads as asking the building task to edit its own entry, and creel-core's decision record cited it to do so.
Every Plan writer, darn writer and rework session loads `CLAUDE.md`, as D3 and D4 rely on, so rework and prior-plan generations see the rule with no stencil of their own.

### D2: The Discussion stencil forbids a decision that edits the task's own entry

**Decision:** `contracts/stencils/loom/loom-template-discussion.md` states that no decision asks the run to change this task's own board entry's body or scope, through `lyx board` or otherwise.
**Rationale:** the Plan writer reads only the decision record, so the decision record is where the bad step entered the pipeline.
The chair stencil includes this stencil's text and needs no copy; the advisor stencil makes no decisions and is left unchanged.

### D3: The Plan stencil forbids such a card

**Decision:** `contracts/stencils/loom/loom-template-plan.md` states that no card changes this task's own board entry's body or scope.
The stencil's post-Discussion route (the paragraph opening "A scope addition that arrives after the Discussion") is widened to name a drop of a decided own-entry step: the parent records the drop with `lyx loom decision add` as an `Added after Discussion` entry, the Plan writer plans without the step only once the record carries that entry, and with no parent recorded it stops and reports.
The widened route admits a drop of an own-entry edit only and no other decided step.
**Rationale:** the Plan writer also loads `CLAUDE.md` and could meet an old record; routing through the parent keeps the record the Plan writer's only source of scope, and the edge is bounded to own-entry edits.

### D4: The darn writer gets the same rule

**Decision:** `contracts/stencils/darn/darn-template-write.md` states that no step changes this task's own board entry's body or scope.
When the board entry or the record asks for such a step, the darn writer leaves the step out from the start, because the run lock refuses the edit anyway, and asks its parent to record the drop with `lyx loom decision add`.
Until the record carries that `Added after Discussion` entry, the writer names the omission in the change description's design decisions as an unrecorded parent correction, which the darn-review rubric files as a MEDIUM `decision` finding deferred to the parent.
Once the entry exists, the omission is no deviation from the amended board entry.
It stops and reports only when it has no parent to ask.
The route admits a drop of an own-entry step only and no other step.
**Rationale:** a darn run holds its entry the same way and reads the same `CLAUDE.md` rule.
The darn route leaves the step out at once where the Plan route (D3) waits for the record, because a darn run is one autonomous session that cannot wait on a reply.

### D5: The reviewing rubrics flag a breach

**Decision:** `contracts/stencils/loom/loom-rubric-discussion-review.md`, `contracts/stencils/loom/loom-rubric-plan-review.md` and `contracts/stencils/loom/loom-rubric-darn-review.md` each name, as a finding, a decision, card or darn step that changes the task's own board entry's body or scope.
The Plan-review line accepts a missing step only when the step changes this task's own board entry's body or scope and the decision record carries an `Added after Discussion` entry recording its drop; any other uncarried Decision stays a finding as before.
The darn-review line accepts a step left out through D4's route once the record carries that entry; before then, the omission is an unrecorded parent correction, filed under the rubric's existing rule as a MEDIUM `decision` finding deferred to the parent.
**Rationale:** a reviewer catches what a writer misses.

## Constraints

- `PATTERN-producer-pointer-rule`: each stencil states the rule for its own role and does not paraphrase another producer's format contract.
- `PATTERN-no-denied-recovery`: the new text names no command the agents' settings deny.
- `PATTERN-stencil-ownership`: the stencils stay files read at call time; seeded copies refresh by the normal deploy.
- `PATTERN-markdown-link-integrity`: the `## Documentation lifecycle` heading in `docs/overview.md` keeps its text, since other docs link to its anchor.
- Every written rule says "body or scope", so it forbids no more than the run lock refuses and leaves the landing's status write outside it.
- Semantic line breaks in every edited `.md` file.

## Auto-mode assumptions

- The landing marks the entry done and the board prune removes it, as the board entry states; no landing change is needed.
- The darn writer and the rubrics are in scope as the same defect, though the entry names only cards.

## Open risks

- Runs already in flight keep the old seeded stencils until the next deploy.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - d4-darn-omission-self-contradiction (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-1-review.md; fixer report: _lyx/reviews/plan/round-1-fixer-report.md.
  - darn-review-evidence-clause-conflates-requested-step (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `CLAUDE.md` and `docs/overview.md` no longer say or imply that a design moves out of, or is cut from, the task's own board entry, and say the entry is left to the landing and prune.
- `CLAUDE.md` states that no decision, card or step of a run changes the running task's own board entry's body or scope.
- The loom Discussion, loom Plan and darn write stencils each state that no step of the run changes this task's own board entry's body or scope, and the Plan and darn write stencils route such a decided step through the parent.
- The three rubrics each name such a decision, card or darn step as a finding.
- Every new rule is scoped to the body or scope of the running task's own entry: a status write, a `run_settings` write, a step writing another entry or a change to board-module code is neither forbidden nor flagged.
- The board lock and `lyx board upsert` behave as before.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass, the `contracts/` stencil tests and the docs link-integrity test included.
