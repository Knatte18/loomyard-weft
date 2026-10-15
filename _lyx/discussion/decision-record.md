# Decision record — card-own-entry-edit

## Goal

No step of a task's run is asked to edit that task's own board entry, which the run's board lock refuses.

## Scope

In:

- The documentation-lifecycle wording in `CLAUDE.md` and `docs/overview.md#documentation-lifecycle`.
- A new `PATTERN.md` entry for the rule.
- The loom Discussion stencil, the loom Plan stencil and the darn write stencil.
- The loom Discussion-review, Plan-review and darn-review rubrics.

Out:

- The board lock and `lyx board upsert`: unchanged, per the operator's 2026-10-15 decision.
- A mechanical plan check in `lyx loom validate-plan`: a card is free text, a card naming `lyx board` also fires on every board-module task, and the defect's own wording ("the board body is cut to a pointer") names no command.
- Board writes to other entries, such as filing a follow-up entry, and changes to the board module's code; they stay allowed.

## Decisions

### D1: The doc rule says the task's own entry is left alone

**Decision:** `CLAUDE.md`'s "Docs land in the same commit" rule and the `docs/overview.md` documentation-lifecycle bullet say that the task building a design writes it into the package's `doc.go` and never edits its own board entry: the landing marks the entry done and the board prune removes it, so no pointer is left behind.
This amends the full statement that `PATTERN-documentation-lifecycle` points at; the entry's own one-line text and its background file stay as they are, since they say only where a built design lives.
**Rationale:** the current sentence "a design whose code has landed moves from the board entry into `doc.go`" reads as asking the building task to edit its own entry, and creel-core's decision record cited it to do so.

### D2: A PATTERN entry states the rule

**Decision:** `PATTERN.md` gains an entry under "Shed and loom", e.g. `PATTERN-own-entry-untouched` — Planning or doing a task's work: no decision, card or step edits the running task's own board entry; its landing retires it.
It has no test and no background file.
**Rationale:** the rule binds several roles, and `CLAUDE.md` puts a new cross-cutting invariant into PATTERN; every spawned agent gets the PATTERN overview inlined, so rework and prior-plan generations see it with no stencil of their own.

### D3: The Discussion stencil forbids a decision that edits the task's own entry

**Decision:** `contracts/stencils/loom/loom-template-discussion.md` states that no decision asks the run to edit this task's own board entry, through `lyx board` or otherwise.
**Rationale:** the Plan writer reads only the decision record, so the decision record is where the bad step entered the pipeline.
The chair stencil includes this stencil's text and needs no copy; the advisor stencil makes no decisions and is left unchanged.

### D4: The Plan stencil forbids such a card

**Decision:** `contracts/stencils/loom/loom-template-plan.md` states that no card edits this task's own board entry.
A decision that asks for one goes through the stencil's existing route for a scope change after Discussion: the Plan writer asks its parent to record the drop and plans without the step once the record carries it, and with no parent recorded it stops and reports.
**Rationale:** the Plan writer also loads `CLAUDE.md` and could meet an old record; routing through the parent keeps the record the Plan writer's only source of scope, and the edge is bounded to own-entry edits.

### D5: The darn writer gets the same rule

**Decision:** `contracts/stencils/darn/darn-template-write.md` states the rule for its own steps.
**Rationale:** a darn run holds its entry the same way and reads the same `CLAUDE.md` rule.

### D6: The reviewing rubrics flag a breach

**Decision:** `contracts/stencils/loom/loom-rubric-discussion-review.md`, `contracts/stencils/loom/loom-rubric-plan-review.md` and `contracts/stencils/loom/loom-rubric-darn-review.md` each name, as a finding, a decision, card or darn step that edits the task's own board entry.
The Plan-review line accepts a step dropped through D4's route.
**Rationale:** a reviewer catches what a writer misses.

## Constraints

- `PATTERN-producer-pointer-rule`: each stencil states the rule for its own role and does not paraphrase another producer's format contract.
- `PATTERN-no-denied-recovery`: the new text names no command the agents' settings deny.
- `PATTERN-stencil-ownership`: the stencils stay files read at call time; seeded copies refresh by the normal deploy.
- `PATTERN-markdown-link-integrity`: the `## Documentation lifecycle` heading in `docs/overview.md` keeps its text, since other docs link to its anchor.
- Semantic line breaks in every edited `.md` file.

## Auto-mode assumptions

- The landing marks the entry done and the board prune removes it, as the board entry states; no landing change is needed.
- The darn writer and the rubrics are in scope as the same defect, though the entry names only cards.

## Open risks

- Runs already in flight keep the old seeded stencils until the next deploy.

## Acceptance criteria

- `CLAUDE.md` and `docs/overview.md` no longer say or imply that a design moves out of, or is cut from, the task's own board entry, and say the entry is left to the landing and prune.
- `PATTERN.md` carries the new entry.
- The loom Discussion, loom Plan and darn write stencils each state that no step of the run edits this task's own board entry, and the Plan stencil routes such a decision through its parent.
- The three rubrics each name such a decision, card or darn step as a finding.
- Every new rule is scoped to the running task's own entry: a step writing another entry or changing board-module code is neither forbidden nor flagged.
- The board lock and `lyx board upsert` behave as before.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass, the `contracts/` stencil tests and the docs link-integrity test included.
