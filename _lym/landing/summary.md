# Rank what makes runs slow and what would make them faster

Adds the study `docs/research/speed/run-slowness-causes-2026-11-06/`, with its report, scripts and tables, and its line in `docs/research/README.md`.
It reads recorded data only: no agent ran and no lym code changed.

## What the report finds

- **Gates are about half of a darn run, and they grew.**
  Tier wall time plus slot wait went from 13.6 minutes a run before 2026-11-01 to 20.6 and 23.6 in the two eras after, and is 18.9 now.
  Three things carry it: a step of 47 to 75 seconds per gate pass at `6c5727df` (one test binary per package), a factor of 1.5 when two or more tier runs share the machine, and more passes of the Darn-Write gate since the cold fixer.
- **Landings queue at Finalize**, which holds the parent lock through its whole gate: 2 to 4 minutes a run on busy days, 16 in one burst.
- **Each row hand-over pays the origin check** added by `f5724733`: 20 seconds a row while pushes held the remote-refs lock, 2 to 4 since `48f17897`.
- **Model time did not grow**, at 12.5 to 14.4 minutes of a darn run and about three quarters of a tapestry run.
  A request takes about 1 second plus 8.2 seconds per 1000 output tokens on Opus and 5.6 on Sonnet, in both periods.
- **Repository size, context size, prompt cache misses and turn count show no effect on run time.**
- Since 2026-11-05 08:03 UTC the median darn run is back at 30.1 minutes against 32.0 before, on a quiet machine.

The report ranks nine changes with an estimated saving each, four of them config alone, and lists six questions the data cannot settle with the cheapest measurement for each.
They are proposals; nothing is built here.

## Design decisions

- **Eras by the commits the data steps at.**
  The earlier study's cut at `f08df88b` is kept, and three more are added at `6c5727df`, `f5724733` and `48f17897`, so each era holds one gate layout and one hand-over cost.
- **Transcripts of 40 runs, the cap.**
  `pick.sh` takes 32 darn code runs in four size bands, 4 before, 2 mid and 2 late per band, and 8 tapestry runs paired by size.
- **Time classes that add up.**
  `timeline.awk` gives each tenth of a second of a row to one class, so model, tools, gates, session starts and idle time sum to the row; 645 rows match their stamps within 1.5 seconds.
- **The earlier study's extraction is reused, not copied.**
  `runs.tsv` comes from `task-runtime-before-after-2026-11-06/extract.sh`, run again; its 277 rows came back identical.
- **Slim committed tables.**
  `turns.tsv` and `tools.tsv` hold no message ids, tool-use ids or command text, and name a session by eight characters.
- **No generated tables file.**
  `analyze.sh` prints every table, and the report holds the ones it argues from.

## To check by hand

- The report's two corrections to `task-runtime-before-after-2026-11-06`: the gate's growth is 7 to 10 minutes once slot wait and every tier run are counted, and the writer's own part of Darn-Write did shrink.
- The savings in "What to do" are estimates from the tables, and each says so.
- `lym report timing` counts a run of the hub's other machine as asleep, because it reads this machine's journal; the report states it under "Limits" and files nothing.
