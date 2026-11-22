# Board

Rendered 2026-11-22 18:22 UTC; every age below is as of then.
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

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Second hardener campaign on fabric, one-repository layout, with the fabric verbs**<br>• Hardener run on the one-repository fabric paths the first campaign could not drive<br>• `hardener: packages ["./internal/fabricengine","./internal/fabriccli","./internal/gitrepo"]`<br>• `loom: notify_parent_on ["Harden-Bouncer","Harden-Burler-Fix"]` | [`harden-fabric-single-repo-verbs`](design-harden-fabric-single-repo-verbs.md) | hardener | Harden-Burler-Review | neve | 2h<br>2h | fabric |

## [P0 Root: entries under no project](project-root.md) (9)

Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T11 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge** | [`orch-autonomy-mode`](design-orch-autonomy-mode.md)<br>[Sketch](decision-orch-autonomy-mode.md) | tapestry · solid | Plan-Burler-Review | neve | 6h<br>6h | orch, landing, loom, pawl, drift |

### Layer A (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Code review asks the parent for held decisions and BLOCKING findings at a spent budget**<br>• A held decision finding gates until the parent rules, and a spent budget with BLOCKING findings open escalates to the parent instead of auto-accepting<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`code-bouncer-asks-parent`](design-code-bouncer-asks-parent.md) | darn · light | T11 | 19m<br>16m | burler, loom, darn |
| T2 | **Per-task beam settings: drop the member-line exception and fix the open findings**<br>• land_notice off means no line for beam members too, and the six review findings per-task-settings-one-lookup landed open are fixed<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`per-task-settings-fixups`](design-per-task-settings-fixups.md) | darn · light | T11 | 18m<br>16m | dobby, config, reconcile, board |
| T4 | **Later run transitions still wait for a records push to origin**<br>• After the start rows, every transition still waits 10 to 20 s for a push; push only where a takeover needs it. | [`later-transition-push-wait`](design-later-transition-push-wait.md)<br>[Sketch](decision-later-transition-push-wait.md) | raddle | T5<br>T11 | 1h<br>0m | batten, fabric |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **A beam pull request description is rewritten after each member landing**<br>• After each member landing under a recorded pull request, a describer agent rewrites the beam change description and lym beam describe updates the pull request; no orch is asked<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`beam-pr-describes-itself`](design-beam-pr-describes-itself.md) | darn · light | T2 | 18m<br>4m | dobby, landing |

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 29m<br>29m | burler, fabric |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 13d<br>13d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>1h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
