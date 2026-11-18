# P0 Root: entries under no project

Rendered 2026-11-18 12:32 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **Result checks in Code-Review, loosened-check rulings in Drift, and PR-Review send-backs to Code-Review** | [`review-result-checks`](design-review-result-checks.md)<br>[Sketch](decision-review-result-checks.md) | tapestry · solid | Webster | neve | 2h<br>1h | stencil, drift, fabric, burler, landing, loom, shed, testkit, docs |
| T9 | **Claude accounts per worktree: pick, pins, headroom, transcript carry and switch on a usage limit** | [`claude-accounts-per-worktree`](design-claude-accounts-per-worktree.md)<br>[Sketch](decision-claude-accounts-per-worktree.md) | tapestry · solid | halted · Plan-Bouncer | neve | 2h<br>1h | claude, shuttle, reed, orch, batten, cwd, config, tools |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 18h<br>17h | board, ide, fabric |
| T6 | `draft` **Where the Claude quota goes, and why it burns faster now than two weeks ago**<br>• Weekly Claude usage W44 to W47 by role, model, hub, recipe and session kind; name what drove the rise and the cuts<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`token-spend-weekly-rise`](design-token-spend-weekly-rise.md) | darn · docs |  | 29m<br>29m | claude, report |

## Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 17h<br>17h | shuttle |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **Clean-room rewrite of the most patched modules, landing first**<br>• Rewrite a module and its tests from a written spec alone, interfaces fixed; pilot on landingshed | [`clean-room-module-rewrite`](design-clean-room-module-rewrite.md) | 33m<br>33m | landing |
| E3 |  | **The comment check costs more than the plan review**<br>• comment-lead and its Haiku forks cost $335 over two weeks, 8 % of all run spend; find why and cut it | [`comment-lead-cost`](design-comment-lead-cost.md) | 2m<br>2m | lint, burler |
| E4 |  | **The Plan writer explorer forks cost more than the writer and show no gain**<br>• Drop the Plan writer exploration subagents; they cost about $6.50 per tapestry run | [`plan-writer-explorer-forks`](design-plan-writer-explorer-forks.md) | 1m<br>1m | loom, stencil |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>4h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
