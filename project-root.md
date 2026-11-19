# P0 Root: entries under no project

Rendered 2026-11-19 11:31 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (8)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **pr-findings fails: NewFinalize Deps.OpenFabric must not be nil**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pr-findings-nil-open-fabric`](design-pr-findings-nil-open-fabric.md) | darn · light | Darn-Write | neve | 9m<br>9m | loom, landing |
| T4 | **gatefixer: gatefix-body-darn prompt fails on missing include loom-fragment-loosened-check**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`gatefix-darn-missing-include`](design-gatefix-darn-missing-include.md) | darn · light | Code-Burler-Fix | neve | 2h<br>40m | loom, stencil |
| T7 | **A docs darn can land with an empty diff and its report only in the change description**<br>• Land accepts an empty diff silently and dobby reports main tip as the landing; a docs darn should commit its report under docs/research<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`empty-landing-not-flagged`](design-empty-landing-not-flagged.md) | darn · light | starting · Seed-IDE | neve | 15h<br>2h | landing, darn |
| T9 | **Burler review round fails hard on invalid review-file YAML frontmatter**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`review-frontmatter-hard-fail`](design-review-frontmatter-hard-fail.md) | darn · light | landing · Finalize | neve | 2h<br>40m | burler, shed |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **Loop ends without envelope or recorded exit after repeated 'session state disagrees' warnings**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`loop-ends-without-envelope`](design-loop-ends-without-envelope.md) | darn · light | T9 | 2h<br>40m | shuttle, shed |
| T13 | **Settings to route code review and drift judge rulings to the run's parent**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`parent-review-code-and-drift`](design-parent-review-code-and-drift.md) | darn · solid | T9 | 2h<br>14m | loom, drift, burler |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired** | [`hardener-campaign`](design-hardener-campaign.md)<br>[Sketch](decision-hardener-campaign.md) | tapestry · solid | T4<br>T9<br>T8<br>T13 | 1h<br>14m | shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>23m | board, ide, fabric |

## Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

### Features (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F2 |  | **Send a run agent questions for the operator to the run parent instead**<br>• A setting that routes every question a run agent would ask the operator in its pane to the parent by creel, with the answer delivered back | [`agent-questions-to-parent`](design-agent-questions-to-parent.md) | 2h<br>2h | loom, shuttle |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>1d | batten, loom, board |
| F4 |  | **Mail the run parent when the run reaches a named row**<br>• A subscription verb or run_settings key so lym mails the parent when a run reaches a named producer, in place of polling | [`parent-notify-on-row`](design-parent-notify-on-row.md) | 2h<br>2h | loom, shed |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
