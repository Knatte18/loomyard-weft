# Board

Rendered 2026-11-15 12:04 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-adopt-revision.md">P1 Revise adopt and the functions that move a hub between machines</a> (2)</strong></summary>

[`adopt-revision`](design-adopt-revision.md) · Age 22m · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Map every piece of state lym keeps, where it lives and whether git holds it** | [`lym-state-map`](design-lym-state-map.md) | darn · docs |  | 26m<br>22m | batten, tender, dobby, orch, fabric, loom |
| T4 | `draft` **Redesign adopt on one FSM of run state, pushed to GitHub at all times** | [`adopt-redesign`](design-adopt-redesign.md) | motif | T3 | 2d<br>21m | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (3)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 11d · Owner `neve`

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
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 9d<br>5d | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 6d<br>4d | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 12d<br>7d | shuttle, codex, loom |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (2)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 5d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Measure whether coverage maps pay off: incremental upkeep time, load on other runs, gate time saved** | [`testmap-payoff-measure`](design-testmap-payoff-measure.md) | darn · docs | Seed-Child | neve | 0m<br>0m | gate, testkit |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 | H | **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | 5d<br>2d | testkit, gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (27)</strong></summary>

Owner `neve`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (4)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, the simple design: a fresh session picks by weight and headroom, a resume finds its transcript**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif |  | 12d<br>3d | claude, shuttle, config |
| T2 | `draft` **Measure what coverage-map upkeep costs, check how its state is shown, and write the operator guide**<br>• What the upkeep passes have cost, whether one call shows the maps state, and a short guide to using and keeping the maps | [`testmap-upkeep-cost-and-guide`](design-testmap-upkeep-cost-and-guide.md) | darn · docs | T5 | 2d<br>2d | gate, testkit, dobby |
| T5 | `draft` **Keep the coverage maps in each pair's weft branch, committed and pushed by lym gate testmap** | [`testmap-maps-in-weft`](design-testmap-maps-in-weft.md)<br>[Sketch](decision-testmap-maps-in-weft.md) | tapestry · solid | T6 | 2d<br>0m | gate, testkit, dobby |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 2d<br>2d | slack, creel, orch |

### Notes (23)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 5d<br>5d | gate, fabric |

#### Bugs (8)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B4 |  | **A pause holds operator verbs, not only runs: the network gate covers every git and GitHub call** | [`pause-holds-operator-verbs`](design-pause-holds-operator-verbs.md) | 30m<br>29m | tender, git, batten, selfreport |
| B5 |  | **Agent prompt gives no way to run slow golden tests by hand; gate skips them silently** | [`gate-skipped-tests-silent`](design-gate-skipped-tests-silent.md) | 30m<br>30m | gate, dotnet, stencil |
| B6 |  | **lym gate testmap refusal's way forward names a path that also refuses** | [`testmap-refusal-names-weft`](design-testmap-refusal-names-weft.md) | 30m<br>30m | gate, cwd |
| B7 |  | **PR-Review deadlocks after a failed pr-review gate when continue is set** | [`pr-review-failed-gate-deadlock`](design-pr-review-failed-gate-deadlock.md) | 30m<br>30m | landing |
| B8 |  | **Body edits on a discussed task never reach its run** | [`discussed-body-edits-lost`](design-discussed-body-edits-lost.md) | 30m<br>30m | board, loom |
| B9 |  | **Worktree .scratch is a symlink into another worktree's .lym/scratch, breaking relative project references** | [`scratch-symlink-other-worktree`](design-scratch-symlink-other-worktree.md) | 30m<br>30m | fabric |
| B10 |  | **Flaky gatecli tests fail the first verify run and pass on rerun** | [`gatecli-flaky-tests`](design-gatecli-flaky-tests.md) | 30m<br>30m | gate |
| B11 |  | **beam merge-parent reuses a completed merge-in slug, so dobby refuses to start the second merge-in** | [`merge-parent-reuses-slug`](design-merge-parent-reuses-slug.md) | 30m<br>30m | dobby, workspaces |

#### Enhancements (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 7d<br>7d | heddle |
| E2 |  | **Raddle cuts darn families for size and concurrency**<br>• Fewer, larger darn parts with a short dependency chain, so a family pays fewer fixed costs and runs more parts at once | [`raddle-darn-sizing`](design-raddle-darn-sizing.md) | 6d<br>3d | shed, loom, darn |
| E3 |  | **Narrow the coverage selector's non-body miss by following the names a changed declaration is referenced by** | [`selector-non-body-miss`](design-selector-non-body-miss.md) | 3d<br>3d | testkit, gate |
| E5 |  | **Darn's Code-Review reviewer should default to Opus, not Sonnet** | [`darn-code-reviewer-opus-default`](design-darn-code-reviewer-opus-default.md) | 30m<br>30m | darn, profiles, loom |
| E6 |  | **Prod code holds test-only functions: 156 unreachable from main; add a PATTERN rule** | [`test-only-functions-in-prod`](design-test-only-functions-in-prod.md) | 30m<br>30m | pattern |
| E7 |  | **Darn prompt forbids gate test, so a task from a reported test failure cannot cheaply check whether it still reproduces** | [`darn-reproduce-reported-failure`](design-darn-reproduce-reported-failure.md) | 30m<br>30m | darn, stencil |

#### Features (8)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F2 |  | **One desktop launcher brings back every hub and beam that was in flight after a crash or reboot**<br>• A lym verb and a .desktop launcher that start each live hub orch and reopen the reed windows | [`machine-restart-shortcut`](design-machine-restart-shortcut.md) | 6d<br>3d | orch, reed, termwindow, shortcut |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 4d<br>4d | config, fabric |
| F5 |  | **A stop button in the prime orch session that takes the whole hub down, behind a yes/no** | [`hub-stop-button`](design-hub-stop-button.md) | 30m<br>30m | reed, orch, tender |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>9d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 5w<br>10d | board, ide, orch |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 5w<br>9d | webster |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>9d | burler, shed, orch, quarry, report |

</details>
