# beam describe: drop the project-decision log from the beam PR description

`beam-describe-no-decision-log` · Note · enhancement, landing, workspaces

Imported from [issue #90](https://github.com/Knatte18/loomyard/issues/90).

## Seen

`lym beam describe <slug>` writes a "Project decisions" section listing every entry `lym loom decision add` added to the project record, each headed "Added after Discussion (operator|parent, <date>)".
In hub `dop`, beam `loomyard-setup` (PR NORCE-DrillingAndWells/DrillOpPlan#30), the operator found that list irrelevant to every reader of the pull request: it is a history of what was decided along the way, partly duplicated and partly superseded (two "No drift baseline" entries, a CLAUDE.md bound replaced twice).
What a reader needs is the result: what the beam ended up with. The per-member PR list under "Changes" is useful and should stay, so a reviewer can follow the change step by step.

There is no way to leave the section out: the verb owns `summary.md`, and the orch may only change the title and brief.

## Workaround

The orch rewrote the beam's brief as an outcome summary. The decision log is still in the PR body.

## Expected

The beam description holds the outcome (title, brief) and the member PR list, without the decision-record log.
If the record should feed the description at all, it should be the record's standing result (goal, bounds, constraints as they now stand), never the chronological list of decisions.


## Issue comment, 2026-11-08

## Operator's design (hub `dop`, 2026-11-08)

The operator wants a different design than trimming the generated text:

- The beam's change description is written by the orch alone, not assembled by `lym beam describe` from the record.
- lym gives the orch clear instructions for how to write it: lead with the result, what the beam ended up with; keep the member PR list so the change can be followed step by step; leave out the history of decisions made along the way.
- The orch may change the description freely, as the operator directs, without going through the entry's title and brief.
- `lym beam describe` (or its successor) then validates and records the orch's text, pushes it to the PR and stamps it into the landing record, instead of owning and rewriting the file.

This replaces the "Expected" section above: the point is not a smaller generated description, but an orch-written one under lym's instructions.
