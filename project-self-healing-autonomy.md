# P3 Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack

Rendered 2026-11-28 09:47 UTC; every age below is as of then.
Under: [Board](README.md).

[`self-healing-autonomy`](design-self-healing-autonomy.md) · Age 7m · Owner `neve`

Groups the entries filed from the Sketch `self-healing-runs`: stop-settlement-table, autonomy-unattended-level, loom-verbs-await-park, stop-halt-count, dobby-pause-lift-resume, driver-settles-stops, arbiter-budget-escalations, severe-halt-slack-post, idle-notice-correct-waits, dobby-stall-chase, orch-unattended-rules.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (8)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | `draft` **The settlement table of stop classes, loom verbs that wait for the driver to park, and the driver's halt count** | [`stop-settlement-table`](design-stop-settlement-table.md)<br>[Sketch](decision-stop-settlement-table.md) | darn · solid | T21 | 7m<br>0m | shed, loom, batten |
| T21 | `draft` **beam.autonomy takes false, true or unattended, and every reader reads the level** | [`autonomy-unattended-level`](design-autonomy-unattended-level.md)<br>[Sketch](decision-autonomy-unattended-level.md) | darn · solid | T9 | 7m<br>7m | dobby, config, cwd, board, orch, slack, loom |
| T32 | `draft` **The dobby resumes a run handed back for a pause once the pause lifts, and never a run the orch holds** | [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)<br>[Sketch](decision-dobby-pause-lift-resume.md) | darn · light | T10<br>T2 | 7m<br>0m | dobby, tender |
| T33 | `draft` **The driver and the arbiter settle known stops and budget escalations themselves within the table's bound** | [`driver-settles-stops`](design-driver-settles-stops.md)<br>[Sketch](decision-driver-settles-stops.md) | darn · solid | T12<br>T11<br>T1 | 7m<br>0m | shed, board, batten, shuttle, webster, arbiter, loom, stencil |
| T35 | `draft` **The dobby posts each severe halt to Slack, and the relay mentions the operator on a severe-halt post** | [`severe-halt-slack-post`](design-severe-halt-slack-post.md)<br>[Sketch](decision-severe-halt-slack-post.md) | darn · light | T33<br>T32 | 7m<br>7m | dobby, batten, slack |
| T36 | `draft` **The idle and held notices skip correct waits and name what an agent waits on** | [`idle-notice-correct-waits`](design-idle-notice-correct-waits.md)<br>[Sketch](decision-idle-notice-correct-waits.md) | darn · light | T35<br>T19 | 7m<br>7m | dobby, batten, creel, shuttle, loom |
| T37 | `draft` **The dobby chases stalls the orch leaves unsettled, reminds of severe posts, and warns when autonomy cannot reach the operator** | [`dobby-stall-chase`](design-dobby-stall-chase.md)<br>[Sketch](decision-dobby-stall-chase.md) | darn · light | T36 | 7m<br>7m | dobby, batten, config |
| T38 | `draft` **The orch role texts gain the unattended level's rules and the glossary names the level** | [`orch-unattended-rules`](design-orch-unattended-rules.md)<br>[Sketch](decision-orch-unattended-rules.md) | darn · docs | T37 | 7m<br>0m | orch, heddle |
