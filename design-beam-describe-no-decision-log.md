# beam describe: drop the project-decision log from the beam PR description

`beam-describe-no-decision-log` · Task · enhancement, landing, workspaces · draft

Imported from [issue #90](https://github.com/Knatte18/loomyard/issues/90).

## Seen

`lym beam describe <slug>` writes a "Project decisions" section listing every entry `lym loom decision add` added to the project record, each headed "Added after Discussion (operator|parent, <date>)".
In hub `dop`, beam `loomyard-setup`, the operator found that list irrelevant to every reader of the pull request:
it is a history of what was decided along the way, partly duplicated and partly superseded.
The per-member PR list under "Changes" is useful and stays.
There is no way to leave the section out: the verb owns `summary.md`, and the orch may only change the title and brief.

## Change (operator, 2026-11-08)

A quick fix, urgent for the work machine:
`lym beam describe` no longer writes the "Project decisions" section.
The description holds the title, the brief and the member PR list under "Changes".
The code is in `internal/landingshed/beamdescribe.go`; its tests and any doc naming the section follow.

The larger design, an orch-written description under lym's instructions, is a separate note (`orch-written-beam-description`) and is out of scope here.
