# The orch writes a beam PR description under lym instructions; beam describe validates and pushes it

`orch-written-beam-description` · Note · feature, landing, workspaces, orch

## Seen (operator, hub `dop`, 2026-11-08, on issue #90)

`lym beam describe` assembles the beam's PR description from the entry and the project record, and owns `summary.md`;
the orch can only change the entry's title and brief.
A generated description cannot follow what the operator wants a reader to see.

## Idea (operator)

- The beam's change description is written by the orch alone, not assembled by `lym beam describe` from the record.
- lym gives the orch clear instructions for how to write it:
  lead with the result, what the beam ended up with;
  keep the member PR list so the change can be followed step by step;
  leave out the history of decisions made along the way.
- The orch may change the description freely, as the operator directs, without going through the entry's title and brief.
- `lym beam describe` (or its successor) validates and records the orch's text, pushes it to the PR and stamps it into the landing record, instead of owning and rewriting the file.
