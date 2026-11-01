# tokencount counts darn runs and compares recipes

`tokencount-darn-runs` · Task · enhancement, tools

## Seen (orch, 2026-11-01)

`tools/tokencount` with no slugs counts only finished full tapestry runs, those that reached Finalize under the tapestry recipe.
On 2026-11-01 it skipped the 80 most recent runs, most of them darns, and counted three tapestry runs from 2026-10-31 and 2026-11-01.
The operator prefers parallel darns to one long loom as cheaper and faster, but no report gives a darn's cost beside a tapestry's.

## Idea

tokencount counts darn runs too, with darn's own segments (writer, Code-Review rounds, landing), and can compare recipes, for example the cost per run and per changed line of darns against tapestries over the same period.
