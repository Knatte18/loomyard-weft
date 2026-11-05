# P0 Root: entries under no project

Rendered 2026-11-05 17:15 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | darn · light | Darn-Write | neve | 4d<br>15m | burler, arbiter, dobby, orch |
| T3 | **Find what sessions load at start that they do not need, to cut start tokens**<br>• Measure the start context of the orch and two spawned roles by source, and name what can be left out<br>• `darn: writer opus` | [`startup-token-trim`](design-startup-token-trim.md) | darn · docs | Finalize | neve | 12m<br>11m | claude, orch, shuttle, stencil |
| T5 | **Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs** | [`slack-relay`](design-slack-relay.md)<br>[Sketch](decision-slack-relay.md) | tapestry · solid | Plan-Write | neve | 48m<br>1m | reed |
| T7 | **Drift compares changed values in code against a sealed expectation, with a separate judge**<br>• General Drift rework: code-side diff of any hub capture keyed by line key or number, structured sealed expectation, containment by code, a judge per changed value | [`drift-rework`](design-drift-rework.md)<br>[Sketch](decision-drift-rework.md) | tapestry · solid | Webster | neve | 1d<br>4h | drift, loom, darn, stencil, config |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Dobby reminds the operator on Slack of a run waiting at PR-Review or Sampler-Session** | [`slack-wait-reminders`](design-slack-wait-reminders.md)<br>[Sketch](decision-slack-wait-reminders.md) | darn · light | T5 | 48m<br>48m | reed |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T7 | 5d<br>4h | fabric |

## Notes (20)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | `draft` **Creel mail between hubs with a verified sender and one-way hub trust**<br>• Orchs of different hubs mail each other; the sending daemon confirms the sender, and a hub obeys only the orchs of hubs it lists as trusted | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md)<br>[Sketch](decision-creel-cross-hub-mail.md) | 6d<br>1m | creel, orch |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 1d<br>11m | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>11m | loom, stencil |

### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• Not a problem now; if built, only after F6 Slack relay, as a Slack reminder ping while a PR-Review waits on the operator | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 1d<br>11m | landing, loom, batten |
| B3 |  | **A finished motif whose Sketch the board refuses stays a draft task with no notice to the orch**<br>• The refusal only logs a Warn; the orch learns of it from a tidy report | [`motif-sketch-refusal-silent`](design-motif-sketch-refusal-silent.md) | 10m<br>10m | loom, board, orch |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 3m<br>3m | reed, orch |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 9d<br>4d | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 2d<br>11m | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 5d<br>2d | shed, shuttle, batten |

### Features (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>9d | burler |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>2d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 6d<br>11m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 3d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>11m | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>11m | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 2d<br>2d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>11m | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 11h<br>11h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 11h<br>11h | batten, board, fabric, workspaces |
