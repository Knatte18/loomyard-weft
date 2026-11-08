# The orch writes the beam PR description under lym instructions; beam describe validates and pushes it

`beam-describe-no-decision-log` · Task · enhancement, landing, workspaces, orch · running · Darn-Write

Imported from [issue #90](https://github.com/Knatte18/loomyard/issues/90).

## Seen

`lym beam describe <slug>` assembles the beam's PR description from the entry and the project record and owns `summary.md`;
the orch can only change the entry's title and brief.
Its "Project decisions" section lists every entry `lym loom decision add` added to the project record, each headed "Added after Discussion (operator|parent, <date>)".
In hub `dop`, beam `loomyard-setup`, the operator found that list irrelevant to every reader of the pull request:
it is a history of what was decided along the way, partly duplicated and partly superseded.
The code is in `internal/landingshed/beamdescribe.go`.

## Design (operator, 2026-11-08)

- The orch writes the whole beam description itself; lym no longer assembles it from the entry and the record.
- lym gives the orch clear instructions for what to write:
  - the end result, what the beam ended up with, never the way there;
  - every member PR, listed so a reader can follow each step to the result on its own;
  - never the list of decisions made along the way.
- The orch and the operator may override anything in the description, at any time, without going through the entry's title and brief.
- `lym beam describe` (or its successor) validates and records the orch's text, pushes it to the PR and stamps it into the landing record, instead of owning and rewriting the file.
  It may still offer the member PR list as a starting point for the orch to edit.
