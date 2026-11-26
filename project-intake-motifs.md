# P6 Intake motifs: run watch, landed records, closed area

Rendered 2026-11-26 10:19 UTC; every age below is as of then.
Under: [Board](README.md).

`intake-motifs` · Age 2h · Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T15 | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | motif | Discussion-Bouncer | neve | 3w<br>1h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Sketches (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Built-in run watch: mail the parent on every non-running state and on a gate slot held too long** | [`dobby-run-watch`](design-dobby-run-watch.md)<br>[Sketch](decision-dobby-run-watch.md) | 2h<br>32m | dobby, gate |
| S2 |  | **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records`](design-keep-landed-records.md)<br>[Sketch](decision-keep-landed-records.md) | 2h<br>13m | landing, fabric, report |
