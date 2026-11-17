# Board

Rendered 2026-11-17 08:43 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codexengine.md">P1 codexengine</a> (8)</strong></summary>

[`codexengine`](design-codexengine.md) · Age 18h · Owner `neve`

### Tasks (6)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Every producer renders its prompt and edit directive for the engine its launch resolves to** | [`prompt-engine-per-launch`](design-prompt-engine-per-launch.md)<br>[Sketch](decision-prompt-engine-per-launch.md) | darn · solid | Drift-Expect | neve | 18h<br>18h | codex, shuttle, stencil, heddle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **A Codex session gets Claude's standing items, user setup, instruction files and subagent terms in its developer instructions** | [`codex-standing-instructions`](design-codex-standing-instructions.md)<br>[Sketch](decision-codex-standing-instructions.md) | tapestry · solid | T6 | 18h<br>18h | codex, shuttle, stencil, heddle |

#### Layer B (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T17 | **Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan** | [`codex-wording-engine-blocks`](design-codex-wording-engine-blocks.md)<br>[Sketch](decision-codex-wording-engine-blocks.md) | darn · solid | T13 | 18h<br>18h | codex, shuttle, stencil, heddle |
| T18 | **A Codex session gets the question-tool deny and the batching nudge Claude runs** | [`codex-question-deny-nudge`](design-codex-question-deny-nudge.md)<br>[Sketch](decision-codex-question-deny-nudge.md) | darn · solid | T13 | 18h<br>18h | codex, shuttle, stencil, heddle |

#### Layer C (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T19 | **Each launch writes its manifest, a parity test compares the engines per seat, and lym shuttle manifest prints both** | [`launch-manifest-parity`](design-launch-manifest-parity.md)<br>[Sketch](decision-launch-manifest-parity.md) | tapestry · solid | T17<br>T18 | 18h<br>18h | codex, shuttle, stencil, heddle |

#### Layer D (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T15 | **Codex as darn writer, replayed on a landed task through the real producer** | [`codex-writer-bench`](design-codex-writer-bench.md) | darn · docs | T19 | 1d<br>18h | codex, shuttle |

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 10d<br>18h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 7d<br>18h | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-adopt-build-resplit.md">P4 Merge the remaining adopt-build members into one design and split it again on what landed</a> (6)</strong></summary>

[`adopt-build-resplit`](design-adopt-build-resplit.md) · Age 8d · Owner `neve`

### Tasks (6)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence** | [`machine-registry-holds-catchup`](design-machine-registry-holds-catchup.md)<br>[Sketch](decision-machine-registry-holds-catchup.md) | tapestry · solid | Plan-Burler-Review | neve | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **lym adopt runs in the foreground over pushed state: hold, owner, pull, takeover, check, with --take-hold** | [`adopt-foreground`](design-adopt-foreground.md)<br>[Sketch](decision-adopt-foreground.md) | tapestry · solid | T4 | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Layer B (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **The dobby tears down or reports leftover pairs** | [`leftover-pairs-teardown`](design-leftover-pairs-teardown.md)<br>[Sketch](decision-leftover-pairs-teardown.md) | darn · solid | T8 | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T12 | **The board prints the prime as the root beam `root`**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`board-prints-root-beam`](design-board-prints-root-beam.md)<br>[Sketch](decision-board-prints-root-beam.md) | darn · light | T8 | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

#### Layer C (2)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **lym beam stop --all takes the hub down on this machine, behind a stop button**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hub-stop-all`](design-hub-stop-all.md)<br>[Sketch](decision-hub-stop-all.md) | darn · solid | T9 | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |
| T11 | **lym machine start brings back every hub this machine owns, from desktop launchers**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`machine-start-launchers`](design-machine-start-launchers.md)<br>[Sketch](decision-machine-start-launchers.md) | darn · solid | T9 | 22h<br>22h | batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle, board, reed, termwindow, shortcut |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (2)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 7d · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T14 | **Survey the mutation-confirmed redundancy candidates across every package**<br>• Run the fixed lym gate redundant on every package and measure how many confirmed candidates are false | [`redundancy-confirmed-survey`](design-redundancy-confirmed-survey.md) | darn · docs | Darn-Write | neve | 18h<br>18h | testkit, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid | T14 | 7d<br>18h | testkit, gate |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (12)</strong></summary>

Owner `neve`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **reed: an RC button per agent, and Slack only in the orch's session**<br>• RC turns on per agent with a click, off by default; the Slack buttons show only where their orch lives | [`reed-orch-buttons-in-tasks`](design-reed-orch-buttons-in-tasks.md) | darn · light | landing · Finalize | neve | 42m<br>36m | reed, orch, slack |
| T16 | **Memory guard for every test binary lym runs**<br>• The gate watchdog watches CPU only and only gate test runs; give it a memory trigger over every test binary, and cap the gate tiers | [`gate-tier-memory-cap`](design-gate-tier-memory-cap.md) | darn · solid | Code-Burler-Fix | neve | 2h<br>36m | gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>20h | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 4d<br>4d | slack, creel, orch |

### Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 7d<br>7d | gate, fabric |
| R4 |  | **Bench Code Mode on a darn writer doing a repeated edit** | [`code-mode-darn-bench`](design-code-mode-darn-bench.md) | 19h<br>19h | claude, shuttle |

#### Features (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 1d<br>20h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 6d<br>22h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>11d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>22h | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>11d | burler, shed, orch, quarry, report |

</details>
