# research: Weaver against Webster on three landed plans

Adds `docs/research/weaver/bench-2026-11-27/report.md`, the four review files beside it and an index entry in `docs/research/README.md`.
It is a measurement report; no code changes.

Weaver implemented three plans that Webster had implemented and landed, from each plan's base commit,
and Webster reran one of them at the same time as Weaver.

- Weaver committed every card of all three plans, passed each plan-level verify on its first run and never stopped.
  One of its ten attempts timed out and climbed the ladder.
- On the plan both ran the same day, Weaver took 91m57s against Webster's 89m28s, cost $10.95 against $13.41,
  and drew 10 review findings (1 blocking) against 20 (3 blocking).
- On the other two plans Weaver took 57m56s and 50m52s against recorded Webster runs of 52m29s and 53m36s,
  and cost $12.73 and $8.60 against $17.00 and $14.91.
  Adjusted by the same-day rerun, the time and cost ratios fall further in Weaver's favour,
  but those two runs had less machine load than the pair that set the adjustment.

The report recommends switching Weaver on per task, not as the default yet.
Before it becomes the default it needs an attempt timeout sized to a batch,
deviations and departures in its summary, gate spans on its row,
and a look at the paths the runs did not reach.

Design decisions and limits, all stated in the report:

- Cost is counted from session transcripts, because the usage records leave out the gate fixers.
- The archived decision records given to the runs name findings the recorded runs' reviews left open.
- The tool was rebuilt and the hub's gate config edited while the two same-day runs were going.
- Finding counts compare like with like only for the same-day pair.

## Merge danger

**Door:** two-way
**Blast radius:** docs
