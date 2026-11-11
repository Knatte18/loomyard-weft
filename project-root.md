# P0 Root: entries under no project

Rendered 2026-11-11 08:02 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (13)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Webster: no amend of a pushed card, no lockless work on a notice, and a gate halt that counts and names its failure** | [`webster-run-faults`](design-webster-run-faults.md) | darn · solid | Code-Burler-Review | neve | 1h<br>30m | webster, loom, creel, batten, git, gate |
| T7 | **gate test: card gate and explicit target never reach a second test project when tier0 command lacks a targets placeholder** | [`gate-target-dropped-silently`](design-gate-target-dropped-silently.md) | darn · light | Code-Burler-Fix | neve | 1h<br>30m | gate, dotnet |
| T8 | **creel post to a strand that is not running reports plain success** | [`creel-post-dead-recipient`](design-creel-post-dead-recipient.md) | darn · light | Code-Burler-Fix | neve | 1h<br>30m | creel |
| T20 | **A loop that ends without an envelope records no exit status, so its halt cannot be explained** | [`loop-exit-not-recorded`](design-loop-exit-not-recorded.md) | darn · light | Darn-Write | neve | 2d<br>1h | shed, friction, shuttle |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **Console window titles put the task handle first, with hub and project in brackets** | [`console-title-handle-first`](design-console-title-handle-first.md) | darn · light |  | 1d<br>2m | termwindow |

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Dobby starts a released raddle task whose depends_on has not landed** | [`dobby-raddle-ignores-deps`](design-dobby-raddle-ignores-deps.md) | darn · light | T4 | 1h<br>30m | dobby, board |
| T14 | **Finalize spawns a fixer on a single non-repeating test failure** | [`finalize-flaky-rerun`](design-finalize-flaky-rerun.md) | darn · light | T7 | 1h<br>9m | landing, gate |

### Layer B (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **machinename reads the machine name from machine.json before the hostname rule** | [`machine-name-file`](design-machine-name-file.md)<br>[Sketch](decision-machine-name-file.md) | darn · light | T6 | 1d<br>33m | config, slack |
| T9 | **FABRIC push-failure notice names the worktree root, which fabric push refuses in an anchored hub** | [`push-notice-anchor-dir`](design-push-notice-anchor-dir.md) | darn · light | T6 | 1h<br>30m | fabric |

### Layer C (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Worktree-Create halts on a reused slug path holding only obj/ leftovers** | [`worktree-create-build-leftovers`](design-worktree-create-build-leftovers.md) | darn · light | T9 | 1h<br>30m | batten, fabric |
| T13 | **lym machine name shows and sets the machine name and sweeps owners across a rename** | [`machine-name-verb`](design-machine-name-verb.md)<br>[Sketch](decision-machine-name-verb.md) | tapestry · solid | T5 | 1d<br>1d | config, slack |

### Layer D (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Remove the hub's _launchers directory: nothing in lym runs its scripts** | [`remove-launchers-dir`](design-remove-launchers-dir.md) | darn · light | T10 | 1h<br>27m | fabric, cleanup |
| T12 | **Slack per orch: a button, machine-first channel names, no presence notice**<br>• An orch gets a Slack channel only when its own slack button is pressed; away stays machine-wide; channel names put the machine first; the presence notice goes. | [`slack-per-orch-activation`](design-slack-per-orch-activation.md) | darn · solid | T13 | 1d<br>3m | slack, reed, creel |

## Notes (22)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 1d<br>1d | gate, fabric |
| R4 |  | **Trial a Sonnet role with an Opus advisor, darn writer first, against Opus all the way** | [`advisor-model-trial`](design-advisor-model-trial.md) | 1h<br>36m | darn, loom, shuttle, claude |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1d | loom, stencil |

### Enhancements (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 3d<br>3d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 2d<br>1d | shed, loom, darn |
| E3 |  | **One land notice: fold the beam-PR landing notice into notify_on_land** | [`one-land-notice`](design-one-land-notice.md) | 1d<br>1d | dobby, batten, workspaces |
| E9 |  | **heddle set/remove should re-render file targets in every live worktree of the hub** | [`heddle-render-all-worktrees`](design-heddle-render-all-worktrees.md) | 1h<br>1h | heddle |
| E10 |  | **heddle file targets: split claude-file and codex-file, add a shared target for both** | [`heddle-split-file-targets`](design-heddle-split-file-targets.md) | 1h<br>1h | heddle |
| E11 |  | **The hub workspace file lists every pair, beams' pairs included, named with their project handle** | [`hub-workspace-all-pairs`](design-hub-workspace-all-pairs.md) | 39m<br>39m | fabric, ide |

### Features (13)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 2d<br>39m | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 2d<br>2d | slack, creel, orch |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 1h<br>1h | config, fabric |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 2d<br>2d | loom, orch, config |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 11d<br>5d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>6d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 8d<br>3d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>5d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>5d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 6d<br>3d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 6d<br>6d | batten, board, fabric, workspaces |
