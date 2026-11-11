# P0 Root: entries under no project

Rendered 2026-11-11 14:36 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (14)

Concrete and claimable; only a task can run.

### Running (6)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T7 | **raddle-propose refuses a note dependency early; a loop exit without an envelope shows its log tail** | [`raddle-dep-and-loop-exit`](design-raddle-dep-and-loop-exit.md) | darn · light | Darn-Write | neve | 5h<br>32m | shed, board, loom |
| T10 | **heddle set/remove should re-render file targets in every live worktree of the hub** | [`heddle-render-all-worktrees`](design-heddle-render-all-worktrees.md) | darn · light | Drift-Expect | neve | 8h<br>3m | heddle |
| T11 | **Remove the hub's _launchers directory: nothing in lym runs its scripts** | [`remove-launchers-dir`](design-remove-launchers-dir.md) | darn · light | Darn-Write | neve | 8h<br>7h | fabric, cleanup |
| T12 | **Slack per orch: a button, machine-first channel names, no presence notice**<br>• An orch gets a Slack channel only when its own slack button is pressed; away stays machine-wide; channel names put the machine first; the presence notice goes. | [`slack-per-orch-activation`](design-slack-per-orch-activation.md) | darn · solid | Darn-Write | neve | 2d<br>6h | slack, reed, creel |
| T17 | **A Sketch or decision record can be edited or replaced, and raddle follows the newer design** | [`decision-record-replace`](design-decision-record-replace.md) | motif | Discussion-Burler-Review | neve | 33m<br>32m | shed, loom, board |
| T19 | **Finalize flaky rerun: test rows for the sink wiring and each rerun guard and outcome** | [`finalize-flaky-test-gaps`](design-finalize-flaky-test-gaps.md) | darn · light | Drift-Baseline | neve | 5h<br>3m | landing, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Adopt takes over without hanging behind a pause, failing on lym files or replaying stale stops** | [`adopt-takeover-fixes`](design-adopt-takeover-fixes.md) | darn · solid | T11 | 1h<br>32m | fabric, tender |
| T8 | **The adopt-check tab closes after the check; reed attach refuses a directory that is no worktree** | [`reed-tab-and-attach-fixes`](design-reed-tab-and-attach-fixes.md) | darn · light | T12 | 4h<br>32m | reed |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **Adopt handover: pin the beam-under-root cover, the hold's skipped steps and the owner archive rows in tests** | [`adopt-handover-test-gaps`](design-adopt-handover-test-gaps.md) | darn · solid | T3 | 5h<br>3m | dobby, fabric, tender |

### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Creel sends an unchanged notice once and drops mail about runs that no longer exist** | [`creel-notice-hygiene`](design-creel-notice-hygiene.md) | darn · light | T5 | 4h<br>32m | creel, dobby, fabric, orch |

### Layer D (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **beam merge-parent reuses the slug of a completed merge-in run** | [`merge-parent-slug-reuse`](design-merge-parent-slug-reuse.md) | darn · light | T6 | 33m<br>32m | workspaces, dobby |
| T16 | **lym shuttle await-forks refused when cwd is a subdirectory of the fabric anchor** | [`await-forks-subdir-cwd`](design-await-forks-subdir-cwd.md) | darn · light | T6 | 33m<br>31m | shuttle, fabric |

### Layer E (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **Tests for the lym verb directory calls and the stops of unconverted steps in an anchored hub** | [`anchored-hub-test-gaps`](design-anchored-hub-test-gaps.md) | darn · light | T16 | 34m<br>3m | fabric |

### Layer F (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T18 | **The hub workspace file lists every pair, beams' pairs included, named with their project handle** | [`hub-workspace-all-pairs`](design-hub-workspace-all-pairs.md) | darn · light | T13 | 7h<br>3m | fabric, ide |

## Notes (22)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2d<br>2d | gate, fabric |
| R4 |  | **Trial a Sonnet role with an Opus advisor, darn writer first, against Opus all the way** | [`advisor-model-trial`](design-advisor-model-trial.md) | 8h<br>5h | darn, loom, shuttle, claude |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>2d | loom, stencil |

### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 3d<br>3d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 3d<br>1d | shed, loom, darn |
| E5 |  | **Refresh the README: outdated parts, less wall of text, the dead Autoboard link and a shorter Lineage**<br>• The README is partly outdated, too long-winded, its Autoboard link is dead, and Lineage no longer fits LoomYard | [`readme-refresh`](design-readme-refresh.md) | 2h<br>2h | docs |
| E6 |  | **Raddle shows each part's recipe and profile as table columns**<br>• Raddle names recipe and profile only in free text; put them in the proposal table | [`raddle-table-recipe-profile`](design-raddle-table-recipe-profile.md) | 5h<br>5h | stencil, shed |
| E10 |  | **heddle file targets: split claude-file and codex-file, add a shared target for both** | [`heddle-split-file-targets`](design-heddle-split-file-targets.md) | 8h<br>8h | heddle |

### Features (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 2d<br>7h | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 3d<br>2h | slack, creel, orch |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 8h<br>6h | config, fabric |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 2d<br>2d | loom, orch, config |
| F6 |  | **Extra reviewer fans at the Discussion perch: parent orch or a named agent** | [`discussion-extra-reviewer-fans`](design-discussion-extra-reviewer-fans.md) | 33m<br>33m | loom, burler, config, orch |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 12d<br>5d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>7d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 9d<br>3d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>5d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>5d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 6d<br>4d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 6d<br>6d | batten, board, fabric, workspaces |
