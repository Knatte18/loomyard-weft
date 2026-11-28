# P3 Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack

Rendered 2026-11-28 13:37 UTC; every age below is as of then.
Under: [Board](README.md).

[`self-healing-autonomy`](design-self-healing-autonomy.md) · Age 3h · Owner `neve`

Groups the entries filed from the Sketch `self-healing-runs`: stop-settlement-table, autonomy-unattended-level, loom-verbs-await-park, stop-halt-count, dobby-pause-lift-resume, driver-settles-stops, arbiter-budget-escalations, severe-halt-slack-post, idle-notice-correct-waits, dobby-stall-chase, orch-unattended-rules.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T21 | **beam.autonomy takes false, true or unattended, and every reader reads the level** | [`autonomy-unattended-level`](design-autonomy-unattended-level.md)<br>[Sketch](decision-autonomy-unattended-level.md) | darn · solid | Darn-Write | neve | 3h<br>3h | dobby, config, cwd, board, orch, slack, loom |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | **The settlement table of stop classes, loom verbs that wait for the driver to park, and the driver's halt count** | [`stop-settlement-table`](design-stop-settlement-table.md)<br>[Sketch](decision-stop-settlement-table.md) | darn · solid | T21 | 3h<br>3h | shed, loom, batten |

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T33 | **The driver and the arbiter settle known stops and budget escalations themselves within the table's bound** | [`driver-settles-stops`](design-driver-settles-stops.md)<br>[Sketch](decision-driver-settles-stops.md) | darn · solid | T12 | 3h<br>3h | shed, board, batten, shuttle, webster, arbiter, loom, stencil |

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T32 | **The dobby resumes lifted pauses, posts severe halts to Slack, keeps quiet on correct waits and chases unsettled stalls** | [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)<br>[Sketch](decision-dobby-pause-lift-resume.md) | darn · solid | T33<br>T2 | 3h<br>3h | dobby, tender, batten, slack, creel, shuttle, loom, config |

## Layer D (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T38 | **The orch role texts gain the unattended level's rules and the glossary names the level** | [`orch-unattended-rules`](design-orch-unattended-rules.md)<br>[Sketch](decision-orch-unattended-rules.md) | darn · docs | T33<br>T32 | 3h<br>3h | orch, heddle |
