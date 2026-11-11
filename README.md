# Board

Rendered 2026-11-11 11:31 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (3)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 7d · Owner `osl-1046`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 5d<br>1d | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 2d<br>3h | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 8d<br>2d | shuttle, codex, loom |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (6)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 1d · Owner `osl-1046`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Research how the coverage maps are kept fresh: three independent brainstorms**<br>• Three agents, two Fable 5.2 and one Opus 5.5, each propose a smarter upkeep of the per-test coverage maps; one report lands<br>• `darn: writer fable[high]` | [`testmap-upkeep`](design-testmap-upkeep.md) | darn · docs |  | 1d<br>1h | testkit, gate, dobby |

#### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **testmap records each test's coverage across every in-module package its binary links** | [`testmap-cross-package`](design-testmap-cross-package.md)<br>[Sketch](decision-testmap-cross-package.md) | darn · solid | T3 | 2h<br>2h | gate, testkit, darn |
| T14 | `draft` **Select an importing package's tests from its coverage map, or name the miss that runs it whole** | [`coverage-test-select`](design-coverage-test-select.md)<br>[Sketch](decision-coverage-test-select.md) | darn · solid | T4 | 2h<br>2h | gate, testkit, darn |
| T15 | `draft` **A coverage gate scope runs changed packages whole and importing packages' selected tests** | [`gate-scope-coverage`](design-gate-scope-coverage.md)<br>[Sketch](decision-gate-scope-coverage.md) | darn · solid | T14 | 2h<br>2h | gate, testkit, darn |

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 1d<br>1d | testkit, gate |
| E12 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 1d<br>4h | dobby, gate, testkit |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (36)</strong></summary>

Owner `osl-1046`

### Tasks (8)

Concrete and claimable; only a task can run.

#### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **One land notice: fold the beam-PR landing notice into notify_on_land** | [`one-land-notice`](design-one-land-notice.md) | darn · light | Code-Burler-Fix | osl-1046 | 1d<br>3h | dobby, batten, workspaces |
| T9 | **FABRIC push-failure notice names the worktree root, which fabric push refuses in an anchored hub** | [`push-notice-anchor-dir`](design-push-notice-anchor-dir.md) | darn · light | Code-Burler-Fix | osl-1046 | 5h<br>3h | fabric |
| T13 | **lym machine name shows and sets the machine name and sweeps owners across a rename** | [`machine-name-verb`](design-machine-name-verb.md)<br>[Sketch](decision-machine-name-verb.md) | tapestry · solid | Plan-Bouncer | osl-1046 | 1d<br>1d | config, slack |
| T20 | **A loop that ends without an envelope records no exit status, so its halt cannot be explained** | [`loop-exit-not-recorded`](design-loop-exit-not-recorded.md) | darn · light | Finalize | osl-1046 | 2d<br>5h | shed, friction, shuttle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (3)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **A load turn retries server_error without a cap, and loom status hides the idle time of a stuck step** | [`load-turn-retry-cap`](design-load-turn-retry-cap.md) | darn · light | T20 | 3h<br>3h | shuttle, loom |
| T10 | **Worktree-Create halts on a reused slug path holding only obj/ leftovers** | [`worktree-create-build-leftovers`](design-worktree-create-build-leftovers.md) | darn · light | T9 | 5h<br>4h | batten, fabric |
| T12 | **Slack per orch: a button, machine-first channel names, no presence notice**<br>• An orch gets a Slack channel only when its own slack button is pressed; away stays machine-wide; channel names put the machine first; the presence notice goes. | [`slack-per-orch-activation`](design-slack-per-orch-activation.md) | darn · solid | T13 | 1d<br>3h | slack, reed, creel |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Remove the hub's _launchers directory: nothing in lym runs its scripts** | [`remove-launchers-dir`](design-remove-launchers-dir.md) | darn · light | T10 | 5h<br>3h | fabric, cleanup |

### Notes (28)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 1d<br>1d | gate, fabric |
| R4 |  | **Trial a Sonnet role with an Opus advisor, darn writer first, against Opus all the way** | [`advisor-model-trial`](design-advisor-model-trial.md) | 5h<br>2h | darn, loom, shuttle, claude |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>2d | loom, stencil |

#### Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Adopt handover: pin the beam-under-root cover, the hold's skipped steps and the owner archive rows in tests** | [`adopt-handover-test-gaps`](design-adopt-handover-test-gaps.md) | 2h<br>2h | dobby, fabric, tender |
| B2 |  | **raddle-propose passes a dependency on a note that Raddle-File refuses**<br>• Validate depends_on targets at raddle-propose, before approval; fix the truncated refusal text | [`raddle-propose-note-deps`](design-raddle-propose-note-deps.md) | 2h<br>2h | shed, board |
| B3 |  | **Creel repeats the same notice many times for an unchanged state**<br>• Dobby and fabric sent identical notices about 25 times during an outage | [`creel-repeat-notices`](design-creel-repeat-notices.md) | 1h<br>1h | creel, dobby, fabric |
| B4 |  | **Creel hands the orch mail about runs and pairs that no longer exist**<br>• A day-old queue of notices about torn-down pairs reached the orch as live mail | [`creel-stale-mail-delivery`](design-creel-stale-mail-delivery.md) | 1h<br>1h | creel, orch |
| B5 |  | **The adopt-check tab stays open and takes space after the check ends**<br>• The tab of a finished adopt check is not closed or shrunk | [`adopt-check-tab-stays`](design-adopt-check-tab-stays.md) | 1h<br>1h | reed |

#### Enhancements (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 3d<br>3d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 2d<br>1d | shed, loom, darn |
| E3 |  | **Finalize flaky rerun: test rows for the sink wiring and each rerun guard and outcome** | [`finalize-flaky-test-gaps`](design-finalize-flaky-test-gaps.md) | 2h<br>2h | landing, gate |
| E6 |  | **Raddle shows each part's recipe and profile as table columns**<br>• Raddle names recipe and profile only in free text; put them in the proposal table | [`raddle-table-recipe-profile`](design-raddle-table-recipe-profile.md) | 2h<br>2h | stencil, shed |
| E9 |  | **heddle set/remove should re-render file targets in every live worktree of the hub** | [`heddle-render-all-worktrees`](design-heddle-render-all-worktrees.md) | 5h<br>5h | heddle |
| E10 |  | **heddle file targets: split claude-file and codex-file, add a shared target for both** | [`heddle-split-file-targets`](design-heddle-split-file-targets.md) | 5h<br>5h | heddle |
| E11 |  | **The hub workspace file lists every pair, beams' pairs included, named with their project handle** | [`hub-workspace-all-pairs`](design-hub-workspace-all-pairs.md) | 4h<br>4h | fabric, ide |

#### Features (13)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 2d<br>4h | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 3d<br>2d | slack, creel, orch |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 5h<br>3h | config, fabric |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 2d<br>2d | loom, orch, config |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 12d<br>5d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>6d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 8d<br>3d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>5d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>5d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 6d<br>3d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 6d<br>6d | batten, board, fabric, workspaces |

</details>
