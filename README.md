# Board

Rendered 2026-11-06 10:04 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Fewer agent turns, fewer tokens](project-turn-savings.md) (1)

`turn-savings` · Age 4d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **What makes runs slow and what would make them faster: deep analysis**<br>• Account for where run time goes, rank the causes and the changes that would cut it, from recorded data<br>• `darn: writer fable[high]` | [`repo-size-run-time`](design-repo-size-run-time.md) | darn · docs | Darn-Write | neve | 7m<br>5m | report, loom |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md) (2)

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 2d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Same real change written by a Codex and a Claude writer, time, cost and quality compared**<br>• Two darn runs of one real lym bug, one writer each; land the better, cancel the other | [`codex-claude-writer-compare`](design-codex-claude-writer-compare.md) | 0m<br>0m | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 3d<br>17h | shuttle, codex, loom |

## [P0 Root: entries under no project](project-root.md) (29)

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Creel mail between hubs: a machine record of hubs, confirmed daemon-to-daemon delivery and replies, per-hub trust and the sender stamp** | [`creel-cross-hub-build`](design-creel-cross-hub-build.md)<br>[Sketch](decision-creel-cross-hub-build.md) | tapestry · solid | Webster | neve | 16h<br>16h | creel, orch |
| T3 | **Trial the session-scope settings in a real launch and re-measure start load**<br>• Check each lever ly-plugin-off-in-other-hubs writes works in Claude Code, and measure one spawned role's and the orch's start load after it | [`session-scope-launch-trial`](design-session-scope-launch-trial.md) | darn · docs | blocked · Worktree-Create | neve | 14h<br>0m | claude, shuttle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light |  | 5d<br>20h | fabric |

### Notes (26)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 1d<br>17h | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>14h | loom, stencil |

#### Bugs (8)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• The Slack reminder ping while a PR-Review waits has landed; only the silence in status views remains, which the operator chose not to build | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 2d<br>23m | landing, loom, batten |
| B2 |  | **Batching hook fires on replay but almost never live** | [`batching-hook-live-gap`](design-batching-hook-live-gap.md) | 14h<br>14h | tools, claude |
| B3 |  | **drift: the judge guard halts on lym's own scope.json write** | [`drift-judge-scope-record-halt`](design-drift-judge-scope-record-halt.md) | 0m<br>0m | drift, loom |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 16h<br>16h | reed, orch |
| B5 |  | **drift: Drift-Recheck halts on a capture's provenance file even when the compare command passed** | [`drift-capture-provenance-halt`](design-drift-capture-provenance-halt.md) | 1h<br>0m | drift, config |
| B6 |  | **drift: config reconcile leaves the retired {skip_file} placeholder in capture and compare commands** | [`drift-retired-skip-file-kept`](design-drift-retired-skip-file-kept.md) | 1h<br>1h | drift, config, reconcile |
| B7 |  | **webster begin-batch reports a delete-before-reference plan ordering finding as plan_conflict: tree_moved** | [`begin-batch-wrong-conflict-kind`](design-begin-batch-wrong-conflict-kind.md) | 1h<br>1h | webster |
| B8 |  | **drift: a judge snapshot gone between resumes halts the run for a hand check** | [`drift-judge-snapshot-gone`](design-drift-judge-snapshot-gone.md) | 0m<br>0m | drift, loom |

#### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Shorten the orch role file, the largest single start cost**<br>• role.md costs 19,390 tokens per read: shorten it, or move rarely used parts to a file the orch reads on demand | [`orch-role-shorter`](design-orch-role-shorter.md) | 13h<br>13h | orch |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 9d<br>14h | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 3d<br>17h | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 5d<br>2d | shed, shuttle, batten |

#### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>10d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 13h<br>13h | reed, shuttle |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>3d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 7d<br>17h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 3d<br>2d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>17h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>17h | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 3d<br>14h | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>17h | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 1d<br>1d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 1d<br>1d | batten, board, fabric, workspaces |
