# Board

Rendered 2026-11-19 11:16 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (2)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 17h · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

### Tasks (2)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | darn · solid | T5<br>T6<br>T10 | 17h<br>58m | fabric, loom, board |
| T11 | `draft` **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | darn · solid | T1<br>T10 | 15h<br>27m | docs |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 2d · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>2d | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-testmap-uses.md">P3 Put the per-test coverage maps to use</a> (0)</strong></summary>

[`testmap-uses`](design-testmap-uses.md) · Age 17h · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records on each code branch, the board on lym-bolt</a> (2)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 15h · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay** | [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md)<br>[Sketch](decision-single-repo-fabric-mode.md) | tapestry · solid | Code-Burler-Fix | neve | 15h<br>15h | fabric, cwd, cli, landing, docs, pattern |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Records-free code identity and pull request views for one-repository hubs** | [`single-repo-code-identity`](design-single-repo-code-identity.md)<br>[Sketch](decision-single-repo-code-identity.md) | tapestry · solid | T5<br>T7 | 15h<br>1h | fabric, cwd, gate, git, weaver, drift, webster, lint, loom, landing, pawl, shed, arbiter |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (14)</strong></summary>

Owner `neve`

### Tasks (8)

Concrete and claimable; only a task can run.

#### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **gatefixer: gatefix-body-darn prompt fails on missing include loom-fragment-loosened-check**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`gatefix-darn-missing-include`](design-gatefix-darn-missing-include.md) | darn · light | Code-Burler-Review | neve | 1h<br>25m | loom, stencil |
| T9 | **Burler review round fails hard on invalid review-file YAML frontmatter**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`review-frontmatter-hard-fail`](design-review-frontmatter-hard-fail.md) | darn · light | Code-Burler-Review | neve | 1h<br>25m | burler, shed |
| T12 | **Intake import drops the issue comments**<br>• lym board intake import keeps only the issue body; comments with further findings are lost unless fetched by hand<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`intake-drops-issue-comments`](design-intake-drops-issue-comments.md) | darn · light | Code-Burler-Fix | neve | 1h<br>25m | board |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (3)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **A docs darn can land with an empty diff and its report only in the change description**<br>• Land accepts an empty diff silently and dobby reports main tip as the landing; a docs darn should commit its report under docs/research<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`empty-landing-not-flagged`](design-empty-landing-not-flagged.md) | darn · light | T5 | 15h<br>1h | landing, darn |
| T8 | **Loop ends without envelope or recorded exit after repeated 'session state disagrees' warnings**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`loop-ends-without-envelope`](design-loop-ends-without-envelope.md) | darn · light | T9 | 1h<br>25m | shuttle, shed |
| T13 | **Settings to route code review and drift judge rulings to the run's parent**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`parent-review-code-and-drift`](design-parent-review-code-and-drift.md) | darn · solid | T9 | 1h<br>0m | loom, drift, burler |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Hardener: the hardener recipe, its round rows, sandbox hubs and live gate, with treadle and crucible retired** | [`hardener-campaign`](design-hardener-campaign.md)<br>[Sketch](decision-hardener-campaign.md) | tapestry · solid | T5<br>T4<br>T9<br>T8<br>T13 | 1h<br>0m | shed, loom, burler, gate, batten, report, board, config, landing, stencil, preflight, pattern, docs |

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>9m | board, ide, fabric |

### Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

#### Features (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F2 |  | **Send a run agent questions for the operator to the run parent instead**<br>• A setting that routes every question a run agent would ask the operator in its pane to the parent by creel, with the answer delivered back | [`agent-questions-to-parent`](design-agent-questions-to-parent.md) | 1h<br>1h | loom, shuttle |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>1d | batten, loom, board |
| F4 |  | **Mail the run parent when the run reaches a named row**<br>• A subscription verb or run_settings key so lym mails the parent when a run reaches a named producer, in place of polling | [`parent-notify-on-row`](design-parent-notify-on-row.md) | 1h<br>1h | loom, shed |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T2 | **Build the mutation gate report step with the trial bound changes**<br>• The mutation trial says build it: 26 of 44 survivors are real missing checks; first spread mutants over functions, skip log and other-OS lines, print one summary line<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`mutation-report-step`](design-mutation-report-step.md) | 2h<br>1h | gate, testmap |

</details>
