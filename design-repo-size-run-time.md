# Does the repository size slow runs, and where did the added fixed cost per run go

`repo-size-run-time` · Note · research, report, loom

## Seen

`docs/research/speed/task-runtime-before-after-2026-11-06` found darn code runs slower since 2026-11-01: active time 31.7 to 45.0 min on average, with the fitted fixed cost per run up from 27 to 41 min.
It could not name the cause.
The operator suspects the repository's own size: Claude takes longer to search and change a larger code base.

Go source on main grew from 28.3 MB in 3116 files (2026-10-27) to 34.7 MB in 3664 files (2026-11-06), about 23%, and markdown from 4.0 to 5.4 MB.

A first look at the report's `runs.tsv`, median darn code run per day, does not show the writer following that growth: write is 20 to 23 min on 2026-10-27 and 28, and 16 to 20 min on 2026-11-03 to 05.
Review rises from about 8 min to 13 to 18 min from 2026-11-03.
Gate wall time rose from 12.1 to 18.3 min per run, which more tests can explain.

## Idea

Measure from the transcripts of landed runs whether size costs time per unit of work, with turns and calls held apart from the task's size:

- greps, reads and lines read per changed line, and grep hits per call, by the repository's size at the run's start;
- context tokens at the first edit, and the time per turn;
- the load each role takes at start from files that grow with the repository, PATTERN and the instruction files among them;
- gate wall time against test count.

Then split the 14 minutes of added fixed cost by row, Record-Write and Drift-Recheck among them, against parallel runs (1.8 to 3.4 in flight) and repository size.
The cap on runs and the choice of sample are set when it is promoted.
