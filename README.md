# Board

Rendered 2026-11-17 18:06 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codexengine.md">P1 codexengine</a> (3)</strong></summary>

[`codexengine`](design-codexengine.md) · Age 1d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T15 | **Codex as darn writer, replayed on a landed task through the real producer** | [`codex-writer-bench`](design-codex-writer-bench.md) | darn · docs | Darn-Write | neve | 1d<br>3h | codex, shuttle |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 11d<br>1d | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 8d<br>3h | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 7h · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>7h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-adopt-build-resplit.md">P4 Merge the remaining adopt-build members into one design and split it again on what landed</a> (1)</strong></summary>

[`adopt-build-resplit`](design-adopt-build-resplit.md) · Age 9d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T11 | **lym machine start brings back every hub this machine owns, from desktop launchers**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`machine-start-launchers`](design-machine-start-launchers.md)<br>[Sketch](decision-machine-start-launchers.md) | darn · solid | Code-Burler-Review | neve | 1d<br>6h | termwindow, dobby |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (1)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 7d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 7d<br>31m | testkit, gate |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (12)</strong></summary>

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md)<br>[Sketch](decision-board-viewer.md) | raddle | Raddle-File | neve | 6w<br>1h | board, ide, orch |
| T7 | **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid | Webster | neve | 4d<br>4d | slack, creel, orch |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>5h | claude, shuttle, config |

### Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |

#### Bugs (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Record Codex edits for the edit-loop watch**<br>• Codex runs record no edits, so the edit-loop watch never notifies their parent | [`codex-edit-loop-recorder`](design-codex-edit-loop-recorder.md) | 5h<br>31m | codex, shuttle |
| B2 |  | **lym gate redundant never mutates a data literal, so a test that pins one is listed redundant**<br>• Mutants change operators only, so a table row or string no other test asserts is no unique kill | [`redundant-mutants-skip-literals`](design-redundant-mutants-skip-literals.md) | 7m<br>7m |  |
| B3 |  | **An old driver fails on board stencils a landing installed mid-run**<br>• A blocked hand-over leaves the old binary rendering new board copies whose includes it lacks | [`stale-driver-new-stencils`](design-stale-driver-new-stencils.md) | 3h<br>3h | stencil, shed |
| B4 |  | **lym gate redundant lists a test redundant when its covers never feed a clause its input**<br>• Negating a condition is killed by any passing test, so a clause no cover exercises alone still counts as covered | [`redundant-clause-inputs-unfed`](design-redundant-clause-inputs-unfed.md) | 6m<br>6m |  |

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **No lym check compares a map entry with a single-test coverage run**<br>• The trial cross-checked by hand; blocks have no line range and funcs have no profile counterpart | [`map-single-test-crosscheck`](design-map-single-test-crosscheck.md) | 3h<br>3h |  |

#### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>1h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
