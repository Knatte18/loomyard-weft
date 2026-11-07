# Board

Rendered 2026-11-07 14:58 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Fewer agent turns, fewer tokens](project-turn-savings.md) (0)

`turn-savings` · Age 5d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P2 Codexengine: every task, trial and note about the Codex engine](project-codex-engine-trial.md) (2)

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 3d · Owner `neve`

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
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 1d<br>1d | codex, shuttle |

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 4d<br>1d | shuttle, codex, loom |

## [P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes](project-fabric-test-speed-work.md) (10)

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 43m · Owner `neve`

### Tasks (10)

Concrete and claimable; only a task can run.

#### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **A slimmer hubforge.CopyHub: packed templates, no walk per copy, two-process binding fold** | [`hubforge-slim-copy`](design-hubforge-slim-copy.md)<br>[Sketch](decision-hubforge-slim-copy.md) | darn · light | Drift-Await | neve | 43m<br>43m | fabric, testkit, git |
| T10 | **internal/hubreconcile fixtures onto registered templates and CopyHub, with the held-lock test at tier 0** | [`hubreconcile-templates`](design-hubreconcile-templates.md)<br>[Sketch](decision-hubreconcile-templates.md) | darn · light | Drift-Expect | neve | 43m<br>43m | fabric, testkit, git |
| T11 | **internal/loomcli beamHub and newBadReedUpFixture onto registered templates** | [`loomcli-templates`](design-loomcli-templates.md)<br>[Sketch](decision-loomcli-templates.md) | darn · light | Drift-Expect | neve | 43m<br>43m | fabric, testkit, git |
| T12 | **internal/battencli beamHub onto a registered template** | [`battencli-templates`](design-battencli-templates.md)<br>[Sketch](decision-battencli-templates.md) | darn · light | Darn-Write | neve | 43m<br>43m | fabric, testkit, git |
| T13 | **Fold the overlapping internal/fabricengine tests the tier 1 report read** | [`fabric-test-folds`](design-fabric-test-folds.md)<br>[Sketch](decision-fabric-test-folds.md) | darn · light | Darn-Write | neve | 43m<br>43m | fabric, testkit, git |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **gitkit and hubforge initialisers write their per-repository git config from Go** | [`git-config-from-go`](design-git-config-from-go.md)<br>[Sketch](decision-git-config-from-go.md) | darn · light | T6 | 43m<br>43m | fabric, testkit, git |
| T14 | **internal/fabricengine anchor and origin fixtures onto registered templates** | [`fabric-anchor-origin-templates`](design-fabric-anchor-origin-templates.md)<br>[Sketch](decision-fabric-anchor-origin-templates.md) | darn · light | T13 | 43m<br>43m | fabric, testkit, git |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **internal/fabricengine live-state arranges and merge-pair layers onto registered templates** | [`fabric-livestate-merge-templates`](design-fabric-livestate-merge-templates.md)<br>[Sketch](decision-fabric-livestate-merge-templates.md) | darn · light | T13<br>T14 | 43m<br>43m | fabric, testkit, git |

#### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T16 | **CopyHub in the NewHub fixtures of fabricengine and adoptcli, and the hook without dirname and basename** | [`copyhub-fixtures-and-hook`](design-copyhub-fixtures-and-hook.md)<br>[Sketch](decision-copyhub-fixtures-and-hook.md) | darn · light | T15 | 43m<br>43m | fabric, testkit, git |

#### Layer D (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Re-measure tier 1 and internal/fabricengine after the fabric test speed tasks land** | [`fabric-test-speed-remeasure`](design-fabric-test-speed-remeasure.md)<br>[Sketch](decision-fabric-test-speed-remeasure.md) | darn · docs | T16<br>T7<br>T10<br>T11<br>T12 | 43m<br>43m | fabric, testkit, git |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P0 Root: entries under no project](project-root.md) (21)

Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **Skills and effort for every role and subagent come from config**<br>• Skills per role in config, gate fixer included; subagents load fitting skills; every effort explicit | [`agent-skills-and-effort`](design-agent-skills-and-effort.md) | darn · solid | Code-Burler-Fix | neve | 5h<br>2h | shuttle, stencil, claude, gate, loom |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Raddle releases the tasks it files, chains shared-package parts against the record order, and shows no dependencies** | [`raddle-order-and-deps-table`](design-raddle-order-and-deps-table.md) | darn · light |  | 44m<br>41m | loom, stencil |

### Notes (19)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 2d<br>1d | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1d | loom, stencil |
| R7 |  | **Re-measure the batching hook once its transcript wait has landed**<br>• Count firings against edit follows and lookup chains again, now that the hook waits for its call to reach the transcript | [`batching-hook-remeasure-wait`](design-batching-hook-remeasure-wait.md) | 2h<br>2h | claude, shuttle |

#### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Model and effort defaults on the claude-settings heddle target** | [`claude-settings-model-effort`](design-claude-settings-model-effort.md) | 1m<br>1m | claude, config |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 10d<br>1d | batten, stencil, config |
| E4 |  | **trusted_hubs lists shortnames, which neither say which repository is meant nor identify it**<br>• List trusted hubs by full repository name instead of shortname | [`creel-trust-by-repo-name`](design-creel-trust-by-repo-name.md) | 6h<br>6h | creel |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 4d<br>1d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 7d<br>3d | shed, shuttle, batten |

#### Features (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>11d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 1d<br>1d | reed, shuttle |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 8d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>3d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 5d<br>1m | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>9m | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 4d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>1d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 2d<br>3h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 2d<br>2d | batten, board, fabric, workspaces |
