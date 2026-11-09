# P0 Root: entries under no project

Rendered 2026-11-09 15:17 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `osl-1046`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Terminal titles read hub: project: task: slug**<br>• Every console window title reads <hub>: <P#>: <T#>: <slug>, with prime, beam and orch forms. | [`terminal-titles-hub-project`](design-terminal-titles-hub-project.md) | darn · light | Finalize | osl-1046 | 1h<br>1h | ide, reed |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (4)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Darn-Write gate fix: rerun only the failed tests between attempts**<br>• Each failed verify attempt reruns the full computed scope; a fixer that gets only the failures and reruns only those would save about 10 minutes per attempt. | [`gate-fix-failures-only`](design-gate-fix-failures-only.md) | darn |  | 46m<br>46m | gate, loom |
| T5 | `draft` **machinename reads the machine name from machine.json before the hostname rule** | [`machine-name-file`](design-machine-name-file.md)<br>[Sketch](decision-machine-name-file.md) | darn · solid |  | 24m<br>24m | config, slack |
| T12 | `draft` **Slack per orch: a button, machine-first channel names, no presence notice**<br>• An orch gets a Slack channel only when its own slack button is pressed; away stays machine-wide; channel names put the machine first; the presence notice goes. | [`slack-per-orch-activation`](design-slack-per-orch-activation.md) | darn · solid | T13 | 1h<br>24m | slack, reed, creel |
| T13 | `draft` **lym machine name shows and sets the machine name and sweeps owners across a rename** | [`machine-name-verb`](design-machine-name-verb.md)<br>[Sketch](decision-machine-name-verb.md) | tapestry · solid | T5 | 24m<br>24m | config, slack |

## Notes (22)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 3h<br>3h | gate, fabric |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>3h | loom, stencil |

### Bugs (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | H | **Tests that compare a measured time to a limit fail on a loaded machine**<br>• TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver took 6.18 s against a 5 s stub delay under load and cost a full gate pass. | [`load-sensitive-timing-tests`](design-load-sensitive-timing-tests.md) | 52m<br>0m | gate, testkit |
| B2 |  | **A loop that ends without an envelope records no exit status, so its halt cannot be explained** | [`loop-exit-not-recorded`](design-loop-exit-not-recorded.md) | 1d<br>1d | shed, friction |
| B3 | H | **The verify gate runs slower tiers after a tier already failed**<br>• A failed tier 0 still ran an 11-minute tier 1; stop at the first failing tier. | [`gate-stop-at-failing-tier`](design-gate-stop-at-failing-tier.md) | 0m<br>0m | gate, darn |
| B4 | H | **A verify gate scopes to nearly the whole suite; scope it by the coverage map**<br>• A diff in a widely imported package reruns tiers 0 to 2 across the suite, 30 to 40 minutes on OSL-1046. | [`gate-scope-by-coverage-map`](design-gate-scope-by-coverage-map.md) | 0m<br>0m | gate, testkit, darn |
| B5 | H | **One new loom config key fails dozens of tests that pin the whole struct**<br>• TestLoadConfig_Loads compares the whole struct in every case, so each added key breaks them all. | [`config-struct-test-per-key`](design-config-struct-test-per-key.md) | 0m<br>0m | loom, config, testkit |
| B6 | H | **The Darn-Write verify gate has no time budget and no notice when it runs long**<br>• verify_attempts bounds attempts, not minutes; a gate stood 35 minutes unannounced. | [`verify-gate-wall-budget`](design-verify-gate-wall-budget.md) | 0m<br>0m | gate, darn |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 1d<br>1d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 1d<br>39m | shed, loom, darn |

### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>13d | burler |
| F2 |  | **One desktop shortcut brings every lym session of the machine back after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 1d<br>1d | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 1d<br>1d | slack, creel, orch |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 1d<br>21h | loom, orch, config |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 10d<br>3d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>5d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 7d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>3d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>39m | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>3d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 4d<br>2d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 4d<br>4d | batten, board, fabric, workspaces |
