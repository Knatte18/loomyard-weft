# P0 Root: entries under no project

Rendered 2026-11-11 16:44 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Creel sends an unchanged notice once and drops mail about runs that no longer exist** | [`creel-notice-hygiene`](design-creel-notice-hygiene.md) | darn · light | Darn-Write | neve | 6h<br>2h | creel, dobby, fabric, orch |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **lym loom decision replace folds a run's record under a lock, cuts the history check at it, and refreshes every stored record** | [`decision-record-replace-verb`](design-decision-record-replace-verb.md)<br>[Sketch](decision-decision-record-replace-verb.md) | darn · solid |  | 1h<br>1h | shed, loom, board |

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **beam merge-parent reuses the slug of a completed merge-in run** | [`merge-parent-slug-reuse`](design-merge-parent-slug-reuse.md) | darn · light | T6 | 2h<br>2h | workspaces, dobby |
| T16 | **lym shuttle await-forks refused when cwd is a subdirectory of the fabric anchor** | [`await-forks-subdir-cwd`](design-await-forks-subdir-cwd.md) | darn · light | T6 | 2h<br>2h | shuttle, fabric |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Tests for the lym verb directory calls and the stops of unconverted steps in an anchored hub** | [`anchored-hub-test-gaps`](design-anchored-hub-test-gaps.md) | darn · light | T16 | 2h<br>2h | fabric |

### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T18 | **The hub workspace file lists every pair, beams' pairs included, named with their project handle** | [`hub-workspace-all-pairs`](design-hub-workspace-all-pairs.md) | darn · light | T13 | 9h<br>2h | fabric, ide |

## Notes (20)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2d<br>2d | gate, fabric |
| R4 |  | **Trial a Sonnet role with an Opus advisor, darn writer first, against Opus all the way** | [`advisor-model-trial`](design-advisor-model-trial.md) | 10h<br>7h | darn, loom, shuttle, claude |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>2d | loom, stencil |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 3d<br>3d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 3d<br>1h | shed, loom, darn |
| E3 |  | **Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by** | [`selector-non-body-miss`](design-selector-non-body-miss.md) | 1h<br>1h | testkit, gate |

### Features (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 3d<br>9h | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 3d<br>22m | slack, creel, orch |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 10h<br>8h | config, fabric |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 3d<br>2d | loom, orch, config |
| F6 |  | **Extra reviewer fans at the Discussion perch: parent orch or a named agent** | [`discussion-extra-reviewer-fans`](design-discussion-extra-reviewer-fans.md) | 2h<br>2h | loom, burler, config, orch |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 12d<br>5d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>7d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 9d<br>48m | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>5d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>5d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 6d<br>4d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 6d<br>6d | batten, board, fabric, workspaces |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T11 | **lym board upsert --record-file replaces a Sketch's stored record, stamped with the body hash promote checks** | [`record-file-upsert`](design-record-file-upsert.md)<br>[Sketch](decision-record-file-upsert.md) | 1h<br>1h | shed, loom, board |
