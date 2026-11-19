# P0 Root: entries under no project

Rendered 2026-11-19 09:54 UTC; every age below is as of then.
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
| T2 | **Build the mutation gate report step with the trial bound changes**<br>• The mutation trial says build it: 26 of 44 survivors are real missing checks; first spread mutants over functions, skip log and other-OS lines, print one summary line<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`mutation-report-step`](design-mutation-report-step.md) | darn · solid | Darn-Write | neve | 50m<br>28m | gate, testmap |
| T4 | **Some test writes a .git into the shared temp directory and fails unrelated packages**<br>• tmuxkit fails every package running beside the writer; find the test that writes it and fix it<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`stray-tmp-git-writer`](design-stray-tmp-git-writer.md) | darn · light | Darn-Write | neve | 14h<br>28m | gate, tools |
| T8 | **A creel post to SLACK reports ok while Slack is disconnected**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`creel-slack-post-silent-when-off`](design-creel-slack-post-silent-when-off.md) | darn · light | Code-Burler-Review | neve | 2h<br>28m | creel, slack |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **A docs darn can land with an empty diff and its report only in the change description**<br>• Land accepts an empty diff silently and dobby reports main tip as the landing; a docs darn should commit its report under docs/research<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`empty-landing-not-flagged`](design-empty-landing-not-flagged.md) | darn · light | T5 | 13h<br>28m | landing, darn |

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>1d | board, ide, fabric |
| T10 | `draft` **Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired** | [`hardener-campaign`](design-hardener-campaign.md)<br>[Sketch](decision-hardener-campaign.md) | tapestry · solid | T5 | 14m<br>14m | shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs |

## Notes (11)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

### Bugs (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **gatefixer: gatefix-body-darn prompt fails on missing include loom-fragment-loosened-check** | [`gatefix-darn-missing-include`](design-gatefix-darn-missing-include.md) | 25m<br>25m | loom, stencil |
| B2 |  | **Loop ends without envelope or recorded exit after repeated 'session state disagrees' warnings** | [`loop-ends-without-envelope`](design-loop-ends-without-envelope.md) | 25m<br>25m | shuttle, shed |
| B3 |  | **Burler review round fails hard on invalid review-file YAML frontmatter** | [`review-frontmatter-hard-fail`](design-review-frontmatter-hard-fail.md) | 25m<br>25m | burler, shed |
| B4 |  | **Intake import drops the issue comments**<br>• lym board intake import keeps only the issue body; comments with further findings are lost unless fetched by hand | [`intake-drops-issue-comments`](design-intake-drops-issue-comments.md) | 24m<br>24m | board |

### Features (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Settings to route code review and drift judge rulings to the run's parent** | [`parent-review-code-and-drift`](design-parent-review-code-and-drift.md) | 25m<br>25m | loom, drift, burler |
| F2 |  | **Send a run agent questions for the operator to the run parent instead**<br>• A setting that routes every question a run agent would ask the operator in its pane to the parent by creel, with the answer delivered back | [`agent-questions-to-parent`](design-agent-questions-to-parent.md) | 24m<br>24m | loom, shuttle |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>1d | batten, loom, board |
| F4 |  | **Mail the run parent when the run reaches a named row**<br>• A subscription verb or run_settings key so lym mails the parent when a run reaches a named producer, in place of polling | [`parent-notify-on-row`](design-parent-notify-on-row.md) | 24m<br>24m | loom, shed |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
