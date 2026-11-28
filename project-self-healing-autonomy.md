# P3 Autonomy settles known run stops itself, so lym runs overnight and only severe halts reach the operator on Slack

Rendered 2026-11-28 09:42 UTC; every age below is as of then.
Under: [Board](README.md).

[`self-healing-autonomy`](design-self-healing-autonomy.md) · Age 2m · Owner `neve`

Groups the entries filed from the Sketch `self-healing-runs`: stop-settlement-table, autonomy-unattended-level, loom-verbs-await-park, stop-halt-count, dobby-pause-lift-resume, driver-settles-stops, arbiter-budget-escalations, severe-halt-slack-post, idle-notice-correct-waits, dobby-stall-chase, orch-unattended-rules.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (11)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T12 | `draft` **The settlement table: stop classes and who settles each, per autonomy level and halt count** | [`stop-settlement-table`](design-stop-settlement-table.md)<br>[Sketch](decision-stop-settlement-table.md) | darn · light |  | 2m<br>2m | shed |
| T21 | `draft` **beam.autonomy takes false, true or unattended, and every reader reads the level** | [`autonomy-unattended-level`](design-autonomy-unattended-level.md)<br>[Sketch](decision-autonomy-unattended-level.md) | darn · solid | T9 | 2m<br>2m | dobby, config, cwd, board, orch, slack, loom |
| T30 | `draft` **lym loom resume and lym loom start wait for the driver to park, with a no-wait option batten passes** | [`loom-verbs-await-park`](design-loom-verbs-await-park.md)<br>[Sketch](decision-loom-verbs-await-park.md) | darn · light | T21 | 2m<br>2m | loom, batten |
| T31 | `draft` **The driver gives each stop an identity, counts halts at a row with lym's retries, and records a halt's four facts** | [`stop-halt-count`](design-stop-halt-count.md)<br>[Sketch](decision-stop-halt-count.md) | darn · solid | T12<br>T30 | 2m<br>2m | shed, loom |
| T32 | `draft` **The dobby resumes a run handed back for a pause once the pause lifts, and never a run the orch holds** | [`dobby-pause-lift-resume`](design-dobby-pause-lift-resume.md)<br>[Sketch](decision-dobby-pause-lift-resume.md) | darn · light | T30<br>T10<br>T2 | 2m<br>2m | dobby, tender |
| T33 | `draft` **The driver settles board claim refusals, failed launches, bounce budgets and gate attempts itself within the table's bound** | [`driver-settles-stops`](design-driver-settles-stops.md)<br>[Sketch](decision-driver-settles-stops.md) | darn · solid | T31<br>T11<br>T1 | 2m<br>2m | shed, board, batten, shuttle, webster |
| T34 | `draft` **The arbiter takes budget escalations, and nothing lands a round with an open blocking finding unattended** | [`arbiter-budget-escalations`](design-arbiter-budget-escalations.md)<br>[Sketch](decision-arbiter-budget-escalations.md) | darn · solid | T33 | 2m<br>2m | arbiter, shed, loom, stencil |
| T35 | `draft` **The dobby posts each severe halt to Slack, and the relay mentions the operator on a severe-halt post** | [`severe-halt-slack-post`](design-severe-halt-slack-post.md)<br>[Sketch](decision-severe-halt-slack-post.md) | darn · light | T33<br>T32 | 2m<br>2m | dobby, batten, slack |
| T36 | `draft` **The idle and held notices skip correct waits and name what an agent waits on** | [`idle-notice-correct-waits`](design-idle-notice-correct-waits.md)<br>[Sketch](decision-idle-notice-correct-waits.md) | darn · light | T35<br>T19 | 2m<br>2m | dobby, batten, creel, shuttle, loom |
| T37 | `draft` **The dobby chases stalls the orch leaves unsettled, reminds of severe posts, and warns when autonomy cannot reach the operator** | [`dobby-stall-chase`](design-dobby-stall-chase.md)<br>[Sketch](decision-dobby-stall-chase.md) | darn · light | T36 | 2m<br>2m | dobby, batten, config |
| T38 | `draft` **The orch role texts gain the unattended level's rules and the glossary names the level** | [`orch-unattended-rules`](design-orch-unattended-rules.md)<br>[Sketch](decision-orch-unattended-rules.md) | darn · docs | T34<br>T37 | 2m<br>2m | orch, heddle |
