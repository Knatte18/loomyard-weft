# P0 Root: entries under no project

Rendered 2026-11-16 12:08 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam** | [`weaver-implementer`](design-weaver-implementer.md)<br>[Sketch](decision-weaver-implementer.md) | tapestry · solid | Drift-Baseline | neve | 1h<br>1h | webster, weaver |
| T6 | **Claude Code Mode for writer sessions, set per role**<br>• Turn on the experimental Code Mode, which batches tool calls in one program, for heavy writer roles only, after a bench | [`claude-code-mode`](design-claude-code-mode.md) | darn · docs | Darn-Write | neve | 2h<br>1h | claude, shuttle, config |
| T16 | **lym gate redundant flags tests as redundant that check different things**<br>• Equal line coverage is taken as redundancy; the verb must also prove the candidate catches no fault its covering tests miss | [`redundancy-judge-asserts`](design-redundancy-judge-asserts.md) | darn · solid | Darn-Write | neve | 44m<br>38m | testkit, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif |  | 13d<br>20h | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 3d<br>3d | slack, creel, orch |
| T13 | `draft` **A filed arras run is refused at Run-Shed and never tears down**<br>• After an arras family filing, batten refuses the run as a grouping project, so it never reaches Land or Worktree-Teardown | [`filed-arras-stuck-at-run-shed`](design-filed-arras-stuck-at-run-shed.md) | darn |  | 1h<br>1h | batten |

## Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 6d<br>6d | gate, fabric |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 8d<br>8d | heddle |

### Features (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F2 |  | **lym retro: one command that audits recent runs from their transcripts and proposes fixes**<br>• Turn the hand-set transcript audits into one command with a fixed lym-scoped question set, routing each finding to a check, a rubric line or a deletion | [`lym-retro`](design-lym-retro.md) | 2h<br>2h | friction, report, tools |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 22h<br>2m | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 5d<br>2h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>10d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>2h | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>10d | burler, shed, orch, quarry, report |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T2 | **Move Webster's batch machinery and run-plan state into a package both implementers import** | [`implementer-shared-run-plan`](design-implementer-shared-run-plan.md)<br>[Sketch](decision-implementer-shared-run-plan.md) | 1h<br>1h | webster, weaver |
