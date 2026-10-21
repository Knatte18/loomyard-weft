# A review perch waits on a reviewer's background shells, and keeps waiting after they are stopped

`review-waits-background-shells` · Note · bug, burler, shuttle

Imported from [issue #796](https://github.com/Knatte18/loomyard/issues/796).

Seen in the quarry hub (Knatte18/quarry, run csharp-alphabet), 2026-10-21.

Plan-Burler-Review finished in about 7.5 minutes and wrote _lyx/reviews/plan/round-1-review.md with verdict APPROVED. The reviewer agent then started two background shell commands (find /). The run waited on them: lyx loom status showed 'waiting: background shells running 21m'. The operator stopped both tasks in the pane ('Stop Task … stopped') and the find processes were gone, yet the run stayed on Plan-Burler-Review with no counter moving.

Expected: a reviewer's background shells started after its verdict file is written never hold the row, or the row stops waiting once the shells have ended; a wait that long surfaces as a stop to the parent rather than a silent hang.
Related: board task run-process-ownership (process ownership); a separate issue covers why the reviewer searched for PATTERN files at all.