## Goal

Releasing a project with lym board set-status releases its draft member tasks

Takes: the board as it is today: Board.SetStatus and Store.SetStatus, validateWrite and the draft rule
Delivers: a null status from `lym board set-status` on a project with no status releases its direct draft member tasks in one write, validated as each member's own release, refusing as a whole on a failure and refusing a project with a run status or done with each member's release as the way forward; Board.SetStatus, Store.SetStatus, upsert and merge release no member
Exports: a Board method the set-status verb alone calls to release a project with its draft members

## Scope

The part changes these packages:

- internal/boardengine (about 150 changed lines)
- internal/boardcli (about 80 changed lines)

## Decisions

### D8 — Releasing a project releases its draft member tasks

Decision: `lym board set-status` with a null status on a project whose own status is empty releases, in the same write, every task whose `project` is that project and whose status is `draft`.
Dobby then starts them in their `depends_on` order, as it starts any released task.
The project keeps the whole decision record, which raddle already stores on a filed project.

Bound:

- The trigger is that verb alone.
  The release lives in the verb's own path or a board method only the verb calls, never in `Board.SetStatus` or `Store.SetStatus`,
  so their other null-status callers, batten cancel's claim clear among them, release no member.
  An upsert or a merge that writes a null status, and D6's filing write, release no member either.
- Only direct member tasks in `draft` are released; a member already released, running or done, a nested project and its members, and entries outside the project are untouched.
- A project whose status is a run status or `done` is refused, naming the release of each member by its slug as the way forward.
- The write validates each released member as its own release would and refuses as a whole when one fails.

Rationale: the operator's one step after an arras run is the release; one command releasing the family keeps it one step.

## Constraints

- `PATTERN-shed-recipe-registry`: arras declares `contract: loom`, and a verb that serves some loom recipes refuses on others by the recipe's rows, never its name (D4's mode, D7's refusal).
- `PATTERN-stencil-ownership`: the new rubric and the raddle stencil's mode text are read at call time through `stencilstore`.
- `PATTERN-sole-parsers`: D5's outcome checks live in `internal/raddleparser`, the proposal's sole parser, which takes the mode as an option.
- `PATTERN-refusal-way-forward`: the refusals of D4, D5, D7 and D8 name their way forward, and a test reaches each.
- `PATTERN-run-records`: post-Discussion decisions are written only by `lym loom decision add`.
- `PATTERN-dobby-runs`: released members start only through their parent's dobby, which D8 relies on and does not change.

## Auto-mode assumptions

- The perch's fixer is cold, not Raddle-Write re-entered (D2).
- A project release releases its draft members at once, and the project keeps the whole record (D8).
- Shape `beam` is refused in an arras run (D5).
- The family's project is the arras entry turned into a project (D6).

## Open risks

## Acceptance criteria

- D8: `lym board set-status` with a null status on a project releases its draft member tasks in one write and leaves released, running and done members and nested projects untouched.
- D8: a project release whose member fails validation releases none, and a project with a run status or `done` is refused.
