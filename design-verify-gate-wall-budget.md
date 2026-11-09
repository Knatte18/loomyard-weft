# The Darn-Write verify gate has no time budget and no notice when it runs long

`verify-gate-wall-budget` · Note · bug, gate, darn

## Seen (operator, 2026-11-09)

The Darn-Write verify gate of `arras-recipe-perch` stood at 35 minutes, and that of `testmap-verb` at about 33 to 40.
`darn.verify_attempts` bounds the number of attempts, but each attempt is a full gate pass, and nothing bounds the time a verify gate may take or tells anyone it is long.

## Idea

Give the verify gate a time budget, and when it is spent, say so to the orch with the gate's failures to date, so the operator sees it before it passes half an hour.
Decide whether a spent budget stops the attempts, hands the run to the orch, or only raises a notice.
Read first what a long verify gate shows today and where a time bound would sit: in the recipe, in `darn.verify_attempts` beside it, or in the gate itself.
