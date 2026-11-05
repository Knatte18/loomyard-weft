# Board

Rendered 2026-11-05 19:01 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Fewer agent turns, fewer tokens](project-turn-savings.md) (1)

`turn-savings` · Age 4d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Re-measure the batching hook and the edit-round rule a week after the hook reaches forks**<br>• run-all.sh on a fresh sample: firing rate, batching after a firing, edit chains split by gate fix against mid-round | [`batching-hook-remeasure`](design-batching-hook-remeasure.md) | 1d<br>1h | tools, claude |

## [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md) (2)

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 1d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B2 |  | **A Codex strand with a wake queued also gets a Mail-is-waiting reminder turn**<br>• Mail posted twice more while a codex queue wake is pending types a reminder that starts a turn of its own, and the queued mail waits behind it | [`codex-idle-wake-extra-reminder`](design-codex-idle-wake-extra-reminder.md) | 3h<br>3h | codex, creel, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 2d<br>1h | shuttle, codex, loom |

## [P0 Root: entries under no project](project-root.md) (29)

Owner `neve`

### Tasks (11)

Concrete and claimable; only a task can run.

#### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Webster refusals: short texts, the right cause and a real way forward** | [`webster-refusal-texts`](design-webster-refusal-texts.md) | darn · solid | Darn-Write | neve | 2m<br>0m | webster, quarry |
| T5 | **Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs** | [`slack-relay`](design-slack-relay.md)<br>[Sketch](decision-slack-relay.md) | tapestry · solid | Webster | neve | 2h<br>1h | reed |
| T7 | **Drift compares changed values in code against a sealed expectation, with a separate judge**<br>• General Drift rework: code-side diff of any hub capture keyed by line key or number, structured sealed expectation, containment by code, a judge per changed value | [`drift-rework`](design-drift-rework.md)<br>[Sketch](decision-drift-rework.md) | tapestry · solid | Parent-Align | neve | 1d<br>5h | drift, loom, darn, stencil, config |
| T9 | **fabric clone --reset removes the hub before refusing a mismatched --shortname** | [`clone-reset-deletes-first`](design-clone-reset-deletes-first.md) | darn · solid | Darn-Write | neve | 2m<br>0m | fabric |
| T10 | **Scope plugins, skills and connectors per role, in every hub**<br>• lym gives each session only the plugins and skills its role needs, ly included in other hubs, and turns off connectors and unused listings | [`ly-plugin-off-in-other-hubs`](design-ly-plugin-off-in-other-hubs.md) | darn · solid | Worktree-Create | neve | 43m<br>0m | shuttle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (3)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **beam merge-parent: the merge-in task can merge, lands as a merge and needs no pull request** | [`merge-parent-lands-right`](design-merge-parent-lands-right.md) | darn · solid | T7<br>T9 | 2m<br>0m | pawl, darn, landing, fabric |
| T6 | **Dobby reminds the operator on Slack of a run waiting at PR-Review or Sampler-Session** | [`slack-wait-reminders`](design-slack-wait-reminders.md)<br>[Sketch](decision-slack-wait-reminders.md) | darn · light | T5 | 2h<br>2h | reed |
| T8 | **Raddle and motif runs start and finish without the orch** | [`raddle-motif-finish-clean`](design-raddle-motif-finish-clean.md) | darn · solid | T7 | 1h<br>0m | batten, dobby, loom, board, orch |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **A parent cannot withdraw a PATTERN change the decision record declares**<br>• lym loom decision add only appends PATTERN bullets; a declared id the parent drops needs a hand edit of the record | [`decision-withdraw-pattern`](design-decision-withdraw-pattern.md) | darn · light | T7<br>T8 | 1h<br>0m | loom, parsers, pattern |

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Creel mail between hubs: a machine record of hubs, confirmed daemon-to-daemon delivery and replies, per-hub trust and the sender stamp** | [`creel-cross-hub-build`](design-creel-cross-hub-build.md)<br>[Sketch](decision-creel-cross-hub-build.md) | tapestry · solid | T5 | 1h<br>1h | creel, orch |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T7 | 5d<br>5h | fabric |

### Notes (18)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 1d<br>1h | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1h | loom, stencil |

#### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• Not a problem now; if built, only after F6 Slack relay, as a Slack reminder ping while a PR-Review waits on the operator | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 1d<br>1h | landing, loom, batten |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 1h<br>1h | reed, orch |

#### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 9d<br>4d | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 2d<br>1h | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 5d<br>2d | shed, shuttle, batten |

#### Features (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>9d | burler |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>2d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 6d<br>1h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 3d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1h | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 2d<br>2d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>1h | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 13h<br>13h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 13h<br>13h | batten, board, fabric, workspaces |
