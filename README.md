# Board

Rendered 2026-11-09 20:17 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-adopt-takeover.md">P1 Adopt as one resumable algorithm that never starts work on its own</a> (3)</strong></summary>

[`adopt-takeover`](design-adopt-takeover.md) · Age 31m · Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Adopt runs as a detached, step-recorded worker that pauses every adopted run and writes the owner before the deploy** | [`adopt-worker-record`](design-adopt-worker-record.md)<br>[Sketch](decision-adopt-worker-record.md) | tapestry · solid | Plan-Burler-Review | neve | 31m<br>31m | batten, tender, dobby, orch, fabric, shortcut |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **The adopting machine's orch and dobby stay up and idle, and the old owner pauses, archives its unpushed tips and exits** | [`adopt-owner-handover`](design-adopt-owner-handover.md)<br>[Sketch](decision-adopt-owner-handover.md) | darn · light | T2 | 31m<br>31m | batten, tender, dobby, orch, fabric, shortcut |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **lym adopt --here behind the lym s shortcut, and lym adopt --archives lists the old owner's archives** | [`adopt-here-archives`](design-adopt-here-archives.md)<br>[Sketch](decision-adopt-here-archives.md) | darn · light | T4 | 31m<br>31m | batten, tender, dobby, orch, fabric, shortcut |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (4)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 6d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (4)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 3d<br>5h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 8h<br>8h | codex, shuttle |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B9 |  | **Codex keeps forcing its bubblewrap sandbox, so astra fails on OSL-1046** | [`codex-bubblewrap-sandbox`](design-codex-bubblewrap-sandbox.md) | 1h<br>1h | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 6d<br>1d | shuttle, codex, loom |

</details>

<details open>
<summary><strong><a href="project-fabric-test-speed-work.md">P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes</a> (0)</strong></summary>

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 2d · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-arras-recipe.md">P4 Arras: one recipe that runs Motif and raddle in a row, with a perch on the split</a> (1)</strong></summary>

[`arras-recipe`](design-arras-recipe.md) · Age 1d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **The shipped rule and docs name arras as the recipe for a tapestry-sized note** | [`arras-shipped-text`](design-arras-shipped-text.md)<br>[Sketch](decision-arras-shipped-text.md) | darn · docs |  | 1d<br>1d | shed, loom, burler, arbiter, board |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T7 | **An arras family filing turns the run's entry into its project and re-points its dependents** | [`arras-family-filing`](design-arras-family-filing.md)<br>[Sketch](decision-arras-family-filing.md) | 1d<br>1d | shed, loom, burler, arbiter, board |

</details>

<details open>
<summary><strong><a href="project-redundancy-check.md">P5 Redundant tests found mechanically: a coverage engine, a map per package, a gate report and its review</a> (1)</strong></summary>

[`redundancy-check`](design-redundancy-check.md) · Age 1d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T11 | **Code review reads the gate's redundancy report, and PATTERN-test-speed names the map, verb and step** | [`review-reads-redundancy`](design-review-reads-redundancy.md)<br>[Sketch](decision-review-reads-redundancy.md) | darn · light | Darn-Write | neve | 1d<br>1d | testkit, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T10 | **An advisory redundancy report step in lym gate test, judging a diff's new tests against the map** | [`gate-redundancy-report`](design-gate-redundancy-report.md)<br>[Sketch](decision-gate-redundancy-report.md) | 1d<br>1d | testkit, gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (28)</strong></summary>

Owner `neve`

### Tasks (9)

Concrete and claimable; only a task can run.

#### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T16 | **Darn verify gate: stop at the first failing tier, rerun only failed tests, a time budget, and a truthful zero-attempt halt** | [`verify-gate-faults`](design-verify-gate-faults.md) | darn · solid | Darn-Write | neve | 5h<br>59m | gate, darn, loom |
| T18 | **Tests that compare a measured time to a limit fail on a loaded machine**<br>• TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver took 6.18 s against a 5 s stub delay under load and cost a full gate pass. | [`load-sensitive-timing-tests`](design-load-sensitive-timing-tests.md) | darn · solid | Darn-Write | neve | 5h<br>1h | gate, testkit |
| T21 | **lym config profiles --set cannot reach a profile's leaf key, and its refusal names the wrong value shape** | [`profiles-set-leaf-key`](design-profiles-set-leaf-key.md) | darn · light | Finalize | neve | 4h<br>1h | config, profiles |

#### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **One new loom config key fails dozens of tests that pin the whole struct**<br>• TestLoadConfig_Loads compares the whole struct in every case, so each added key breaks them all. | [`config-struct-test-per-key`](design-config-struct-test-per-key.md) | darn · light |  | 4h<br>23m | loom, config, testkit |

#### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **machinename reads the machine name from machine.json before the hostname rule** | [`machine-name-file`](design-machine-name-file.md)<br>[Sketch](decision-machine-name-file.md) | darn · light |  | 5h<br>5h | config, slack |
| T12 | `draft` **Slack per orch: a button, machine-first channel names, no presence notice**<br>• An orch gets a Slack channel only when its own slack button is pressed; away stays machine-wide; channel names put the machine first; the presence notice goes. | [`slack-per-orch-activation`](design-slack-per-orch-activation.md) | darn · solid | T13 | 6h<br>5h | slack, reed, creel |
| T13 | `draft` **lym machine name shows and sets the machine name and sweeps owners across a rename** | [`machine-name-verb`](design-machine-name-verb.md)<br>[Sketch](decision-machine-name-verb.md) | tapestry · solid | T5 | 5h<br>5h | config, slack |
| T17 | `draft` **A verify gate scopes to nearly the whole suite; scope it by the coverage map**<br>• A diff in a widely imported package reruns tiers 0 to 2 across the suite, 30 to 40 minutes on OSL-1046. | [`gate-scope-by-coverage-map`](design-gate-scope-by-coverage-map.md) | motif |  | 4h<br>1h | gate, testkit, darn |
| T20 | `draft` **A loop that ends without an envelope records no exit status, so its halt cannot be explained** | [`loop-exit-not-recorded`](design-loop-exit-not-recorded.md) | darn · light |  | 1d<br>1h | shed, friction |

### Notes (19)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8h<br>8h | gate, fabric |
| R6 |  | **Plan writer cost: a solo writer against twill-plan, and an excerpt cut**<br>• The Plan writer is the costliest session type; the open questions are a solo writer against twill-plan and an excerpt cut for the exploration fan | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>8h | loom, stencil |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **The prime dobby leaves a coverage map backlog unbuilt until more landings come** | [`testmap-backlog-drain`](design-testmap-backlog-drain.md) | 39m<br>39m | dobby, gate, testkit |

#### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 2d<br>2d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 1d<br>47m | shed, loom, darn |
| E3 |  | **One land notice: fold the beam-PR landing notice into notify_on_land** | [`one-land-notice`](design-one-land-notice.md) | 4h<br>4h | dobby, batten, workspaces |
| E4 |  | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 1h<br>1h | testkit, gate |

#### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>13d | burler |
| F2 |  | **One desktop shortcut brings every lym session of the machine back after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 1d<br>1d | orch, reed, termwindow, shortcut |
| F3 |  | **Let orchs on two machines talk through a Slack bridge channel**<br>• A shared Slack channel where each machine's bot posts and reads, with the other machine marked as sender, never as the operator | [`slack-machine-bridge`](design-slack-machine-bridge.md) | 1d<br>1d | slack, creel, orch |
| F5 |  | **Let the parent orch review the Discussion decision record before Plan** | [`parent-reviews-decision-record`](design-parent-reviews-decision-record.md) | 1d<br>1d | loom, orch, config |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 10d<br>4d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>5d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 7d<br>2d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4w<br>4d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>47m | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>4d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 4d<br>2d | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 4d<br>4d | batten, board, fabric, workspaces |

</details>
