# Close a task's scope at Code-Review, and check earlier what PRs find

A task's scope can no longer change once its code is under review.
A decision recorded that late may never be judged against the diff or built.
From the recipe's first Code-Review row until its Finalize row, `loom decision add` and `loom decision replace` refuse every caller.
A Code-Review decision finding that the record already asks for can now reach a fixer.
Discussion, the plan, Code-Review and the darn writer also check earlier what otherwise surfaces at pull request review.

Closes #226
Closes #225

## The scope line

- `loomrecipe.InClosedScope` reads the line from the recipe's own rows.
  A recipe with no Code-Review segment, such as arras, has no line.
- A run whose Code-Review segment has run stays closed when it is sent back to an earlier row.
- The one call still taken is an `add` on a `bug` entry, titled `No test seam` or `No bug fixed`,
  that amends no plan, changes no PATTERN entry and drops no Create target.
  It stays open because the red-first check lifts only on a record with one of those titles.
- The refusal writes nothing and tells the caller to file the call as a note for a new task, with `board upsert` and kind note.
  Halts met after the line, such as a plan conflict or a darn writer's stop, name the same route,
  as do the parent's role files and `docs/overview.md`.
- A plan conflict met after the line can take no amendment.
  The way forward is to file the fix as a note and send the run to `Plan-Write` (`Parts-Plan-Write` for a split plan),
  which writes the plan again from the record as it stands.
- Finalize is outside the line, so a conflict its fixer finds with a recorded decision is still settled with `decision add`.

## A decision finding ruled in scope reaches a fixer

- `loom circling continue <slug> --in-scope <finding-id>`, the flag repeated per finding, answers a decision escalation.
  It refuses any other escalation cause, and an id that names no decision-class finding of that round's review.
- The ids are recorded as `in_scope` in the round's circling decision file.
  The next review carries each such finding under a class other than `decision`, so that round's fixer takes it,
  or resolves it without quoting an added entry.
- A new Bouncer key, `scope_closed`, is set on the Code-Review Bouncer row of the tapestry, darn and sampler recipes.
  Their decision escalations name the in-scope ruling and a note instead of `decision add`.
  At the hard cap no further round exists, so they name the note alone.

## Earlier checks

- Discussion: `## Acceptance criteria` names each figure to measure and state with its measured case,
  each doc statement to make true, and each new test's intent and tier.
  The Discussion review flags an omission, and the Describe review reads the criteria as things the description states.
- Plan: a new `gate.yaml` key, `plan_tier0_budget_sec`, bounds the seconds a plan's new and changed tier0 tests may add to one tier0 run.
  The template sets 60, and 0 sets no budget.
  Each card that creates or widens a tier0 test states its estimate,
  and the plan review flags a missing estimate or a sum past the budget.
- Code-Review: a quoted measured figure whose measured case the diff changes is a finding unless it is re-measured or dropped.
  The darn writer is told the same.

## Design decisions

- Operator feedback that arrives after the line, on the pull request or elsewhere, is a note like any other late call.
  There is no `--by operator` exemption: `--by` is an unverified label, so an exemption would reopen the scope to any agent.
- Running the mutation or redundancy step on every C# change is left out.
  Those steps are built on the coverage maps, which are slated for removal.
- The 60-second tier0 budget is a starting value, not a measured one.

## Merge danger

**Door:** two-way
**Blast radius:** loom

After a revert, a `plan_tier0_budget_sec` left in the hub's `gate.yaml` only draws an unknown-key warning.
