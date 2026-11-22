# Board

Rendered 2026-11-22 17:04 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 5d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>1d | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 3d · Owner `neve`

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **Second hardener campaign on fabric, one-repository layout, with the fabric verbs**<br>• Hardener run on the one-repository fabric paths the first campaign could not drive<br>• `hardener: packages ["./internal/fabricengine","./internal/fabriccli","./internal/gitrepo"]`<br>• `loom: notify_parent_on ["Harden-Bouncer","Harden-Burler-Fix"]` | [`harden-fabric-single-repo-verbs`](design-harden-fabric-single-repo-verbs.md) | hardener | T1 | 58m<br>58m | fabric |

## [P0 Root: entries under no project](project-root.md) (9)

Owner `neve`

### Running (3)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **A killed seed leaves board sync naming a false, looping way forward**<br>• After a killed seeding lym, board sync says the board was never seeded and names a --reset clone that may be refused.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`seed-killed-way-forward-loops`](design-seed-killed-way-forward-loops.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | fabric |
| T2 | **Each run transition waits for a push to origin, so a run starts about a minute slower**<br>• Since 71d6ee1a0 every batten row pushes its records before acting; a run start went from ~7 s to ~60 s.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`transition-push-slows-rows`](design-transition-push-slows-rows.md) | darn · light | Code-Bouncer | neve | 17m<br>7m | batten, fabric |
| T3 | **Per-task beam settings use one lookup from the worktree up**<br>• One lookup for per-task beam settings: the task worktree's own value, else its parent's up to the hub; pull_request and land_notice move onto it.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`per-task-settings-one-lookup`](design-per-task-settings-one-lookup.md) | darn · solid | Code-Burler-Fix | neve | 5h<br>5h | dobby, config, board, landing |

### Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **Orch procedure to move a running task by a move note and a fresh run**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-move-running-task`](design-orch-move-running-task.md)<br>[Sketch](decision-orch-move-running-task.md) | darn · docs | T3 | 12m<br>12m | orch, heddle |
| T6 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md) | motif | T2 | 7m<br>2m | batten, fabric |
| T11 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge** | [`orch-autonomy-mode`](design-orch-autonomy-mode.md)<br>[Sketch](decision-orch-autonomy-mode.md) | tapestry · solid | T3 | 5h<br>5h | orch, landing, loom, pawl, drift |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
