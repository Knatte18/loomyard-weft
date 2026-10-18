# lyx report timing leaves out darn runs

`timing-report-darn` · Task · bug, report · running · Publish

## Seen (orch, 2026-10-17)

`lyx report timing --since 2026-10-01` lists 161 runs and none of the darn runs that landed in that window: `gate-wait-cache-cap`, `landing-trailer-text` and `stencilstore-dev-warning-noise`, among others, are absent from `.runs`.
Its phase columns are loom's (setup, discussion, plan, webster, merge, publish-finalize, land), so a darn run's rows (Darn-Write, Darn-Bouncer, Darn-Burler-Review, Darn-Burler-Fix) have no column either.
To compare darn with loom, the orch read each darn run's last batten `status.json` from the records repository's history instead: 29 darn runs, median span from first to last history entry 22 minutes (15 to 31), sleep and waits not taken out, against a loom active-time median of 108 minutes.

## Idea

- `lyx report timing` reports darn runs beside loom runs, tagged with their recipe, with the same sleep and wait accounting.
- Darn's rows map to phases of their own (such as write and review), or the report's phase set comes from the recipe, so a new recipe needs no report change.
- `lyx report convergence` is checked for the same gap.
