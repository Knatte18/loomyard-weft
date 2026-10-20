# A writer edits nothing until every lens fork has returned

`fork-findings-wait-all` · Task · enhancement, stencil, burler, loom · done

## Seen (operator, 2026-10-20)

A writer that spawned its lens forks started editing as soon as the first fork returned, before the rest had reported.
The twill directive (`contracts/stencils/twill/twill-directive.md`) and the fixer self-check (`contracts/stencils/burler/burler-fix-self-check.md`) say "Wait for every fork to return" and then "Judge every finding: merge duplicates across lenses", but neither says outright that nothing is edited before the last fork returns.

Editing early saves little, since the forks run in parallel and the wait is about the slowest fork's time.
It costs the whole view: duplicates across lenses cannot be merged once the first is fixed, a later finding cites text that has since changed, and of two findings proposing different changes at one spot the first applied tends to stay.

## Idea

Both stencils state it as a hard rule: edit nothing until every fork has returned, then judge all findings together, then edit.
The same check applies to any other stencil where a session spawns review forks and acts on their findings.
