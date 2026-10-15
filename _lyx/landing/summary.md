# Keep cards and steps off the running task's own board entry

A planned card once asked the implementer to cut the task's own board entry to a pointer through `lyx board upsert`.
The run's board lock refuses that edit, so the step could not be completed.
The source was the documentation-lifecycle wording: "a design whose code has landed moves from the board entry into `doc.go`" reads as asking the building task to edit its own entry.
The entry needs no edit from the run, because the landing marks it done and the board prune removes it.

The board lock and `lyx board upsert` are unchanged.
The fix is in the rules and stencils that produce such a step.

Closes #653

## What changes

- `CLAUDE.md` and `docs/overview.md#documentation-lifecycle`: the task building a design writes it into the package's `doc.go` and never changes its own entry's body or scope; no pointer is left behind.
  `CLAUDE.md` also states that no decision, card or step of a run changes the running task's own entry's body or scope.
- The loom Discussion stencil forbids a decision that edits the own entry.
- The loom Plan stencil forbids such a card.
- The darn write stencil forbids such a step.
- The Discussion-review, Plan-review and darn-review rubrics each name such a decision, card or step as a finding.

Every rule says "body or scope", so it forbids no more than the lock refuses.
The landing's status write, `run_settings`, writes to other entries and board-module code changes are neither forbidden nor flagged.

## Design decisions

- An already-decided own-entry step is dropped through the parent.
  The Plan stencil's post-Discussion route is widened: the parent records the drop with `lyx loom decision add` as an `Added after Discussion` entry, and the Plan writer plans without the step only once the record carries it.
  With no parent recorded, the writer stops and reports.
- The darn writer cannot wait on a reply, so it leaves the step out at once and asks its parent to record the drop.
  Until the entry exists, it names the omission in the change description as an unrecorded parent correction, which the darn-review rubric files as a MEDIUM `decision` finding deferred to the parent.
  Both routes admit a drop of an own-entry step only.
- No mechanical check in `lyx loom validate-plan`: a card is free text, and a card naming `lyx board` would also fire on every board-module task.
- No `PATTERN.md` entry: the rule binds agent artifacts, and PATTERN states rules for the code.
- The `PATTERN-documentation-lifecycle` entry and its background file are unchanged; they say only where a built design lives.

## To check by hand

- `contracts/stencils/loom/loom-rubric-darn-review.md`, the new own-entry bullet: its evidence clause treats the change description's claim as the evidence for a step, since a `lyx board` write never shows in the committed diff, and a decision-record step counts only as evidence of the request.
  Check that this reads consistently with the darn writer's omission route.
- `contracts/stencils/darn/darn-template-write.md`: the writer's omission wording against the rubric's "unrecorded parent correction" handling, which is the same route stated from both sides.
- Runs already in flight keep the old seeded stencils until the next deploy.

`lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description; run both before pushing.
