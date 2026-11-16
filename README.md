# Board

Rendered 2026-11-16 14:07 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codex-engine-trial.md">P2 Codexengine: every task, trial and note about the Codex engine</a> (4)</strong></summary>

[`codex-engine-trial`](design-codex-engine-trial.md) · Age 12d · Owner `neve`

Real Codex launches through lym shuttle check home, sandbox, hooks, guards, pane, skills and load

### [P1 A Codex session gets exactly what a Claude session in the same role gets](project-codex-parity.md) (5)

[`codex-parity`](design-codex-parity.md) · Age 11m · Owner `neve`

#### Tasks (5)

Concrete and claimable; only a task can run.

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Draft (5)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | `draft` **Every producer renders its prompt and edit directive for the engine its launch resolves to** | [`prompt-engine-per-launch`](design-prompt-engine-per-launch.md)<br>[Sketch](decision-prompt-engine-per-launch.md) | darn · solid | T5 | 11m<br>11m | codex, shuttle, stencil, heddle |
| T13 | `draft` **A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions** | [`codex-standing-instructions`](design-codex-standing-instructions.md)<br>[Sketch](decision-codex-standing-instructions.md) | tapestry · solid | T6 | 11m<br>11m | codex, shuttle, stencil, heddle |
| T17 | `draft` **Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan** | [`codex-wording-engine-blocks`](design-codex-wording-engine-blocks.md)<br>[Sketch](decision-codex-wording-engine-blocks.md) | darn · solid | T13 | 11m<br>11m | codex, shuttle, stencil, heddle |
| T18 | `draft` **A Codex session gets the question-tool deny and the batching nudge Claude runs** | [`codex-question-deny-nudge`](design-codex-question-deny-nudge.md)<br>[Sketch](decision-codex-question-deny-nudge.md) | darn · solid | T13 | 11m<br>11m | codex, shuttle, stencil, heddle |
| T19 | `draft` **Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both** | [`launch-manifest-parity`](design-launch-manifest-parity.md)<br>[Sketch](decision-launch-manifest-parity.md) | tapestry · solid | T17<br>T18 | 11m<br>11m | codex, shuttle, stencil, heddle |

#### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Own entries

The entries directly under this project and under none of its sub-projects.

#### Tasks (1)

Concrete and claimable; only a task can run.

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | `draft` **Codex as darn writer, replayed on a landed task through the real producer** | [`codex-writer-bench`](design-codex-writer-bench.md) | darn · docs | T19 | 19h<br>11m | codex, shuttle |

#### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

##### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 10d<br>3h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 7d<br>19h | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-adopt-build-resplit.md">P4 Merge the remaining adopt-build members into one design and split it again on what landed</a> (7)</strong></summary>

[`adopt-build-resplit`](design-adopt-build-resplit.md) · Age 7d · Owner `neve`

### Tasks (7)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **A transition pushes its records before the run acts, and a takeover entry in the run's records hands the run over** | [`transition-push-takeover`](design-transition-push-takeover.md)<br>[Sketch](decision-transition-push-takeover.md) | tapestry · solid | Webster | neve | 3h<br>2h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence** | [`machine-registry-holds-catchup`](design-machine-registry-holds-catchup.md)<br>[Sketch](decision-machine-registry-holds-catchup.md) | tapestry · solid | T3 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **lym adopt runs in the foreground over pushed state: hold, owner, pull, takeover, check, with --take-hold** | [`adopt-foreground`](design-adopt-foreground.md)<br>[Sketch](decision-adopt-foreground.md) | tapestry · solid | T4 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Layer C (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **The dobby tears down or reports leftover pairs** | [`leftover-pairs-teardown`](design-leftover-pairs-teardown.md)<br>[Sketch](decision-leftover-pairs-teardown.md) | darn · solid | T8 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T12 | **The board prints the prime as the root beam `root`**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-prints-root-beam`](design-board-prints-root-beam.md)<br>[Sketch](decision-board-prints-root-beam.md) | darn · light | T8 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Layer D (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **lym beam stop --all takes the hub down on this machine, behind a stop button**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hub-stop-all`](design-hub-stop-all.md)<br>[Sketch](decision-hub-stop-all.md) | darn · solid | T9 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T11 | **lym machine start brings back every hub this machine owns, from desktop launchers**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`machine-start-launchers`](design-machine-start-launchers.md)<br>[Sketch](decision-machine-start-launchers.md) | darn · solid | T9 | 3h<br>3h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (2)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 6d · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T14 | **Survey the mutation-confirmed redundancy candidates across every package**<br>• Run the fixed lym gate redundant on every package and measure how many confirmed candidates are false | [`redundancy-confirmed-survey`](design-redundancy-confirmed-survey.md) | darn · docs | Darn-Write | neve | 1m<br>1m | testkit, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid | T14 | 6d<br>1m | testkit, gate |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (13)</strong></summary>

Owner `neve`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam** | [`weaver-implementer`](design-weaver-implementer.md)<br>[Sketch](decision-weaver-implementer.md) | tapestry · solid | Plan-Burler-Fix | neve | 3h<br>3h | webster, weaver |
| T16 | **lym retro: one command that audits recent runs from their transcripts and proposes fixes**<br>• Turn the hand-set transcript audits into one command with a fixed lym-scoped question set, routing each finding to a check, a rubric line or a deletion | [`lym-retro`](design-lym-retro.md) | darn · solid | Preflight | neve | 4h<br>1m | friction, report, tools |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>1h | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 3d<br>3d | slack, creel, orch |

### Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 7d<br>7d | gate, fabric |
| R4 |  | **Bench Code Mode on a darn writer doing a repeated edit** | [`code-mode-darn-bench`](design-code-mode-darn-bench.md) | 55m<br>55m | claude, shuttle |

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | 8d<br>8d | heddle |

#### Features (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 1d<br>2h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 5d<br>4h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>10d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>4h | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>10d | burler, shed, orch, quarry, report |

</details>
