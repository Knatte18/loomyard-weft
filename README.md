# Board

Rendered 2026-11-19 07:43 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (3)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 14h · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | 11h<br>11h | docs |
| E5 |  | **Design: simplify interfaces that make every caller handle too much**<br>• For example one result type instead of a four-value return handled at 60 call sites; find such spots per module and fix them one module at a time | [`design-simplify-callers`](design-design-simplify-callers.md) | 14h<br>14h | fabric, landing |
| E6 |  | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | 14h<br>17m | fabric, loom, board |

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 1d · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>1d | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-testmap-uses.md">P3 Put the per-test coverage maps to use</a> (1)</strong></summary>

[`testmap-uses`](design-testmap-uses.md) · Age 13h · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T7 | **Coverage map uses: reader, reach note, failure triage and lym gate mutate** | [`testmap-reach-triage-mutate`](design-testmap-reach-triage-mutate.md)<br>[Sketch](decision-testmap-reach-triage-mutate.md) | tapestry · solid | Drift-Verify | neve | 12h<br>12h | testmap, gate, burler, darn, webster, loom, shed, stencil, pattern |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records on each code branch, the board on lym-bolt</a> (2)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 11h · Owner `neve`

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay** | [`single-repo-fabric-mode`](design-single-repo-fabric-mode.md)<br>[Sketch](decision-single-repo-fabric-mode.md) | tapestry · solid | Webster | neve | 11h<br>11h | fabric, cwd, cli, landing, docs, pattern |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Records-free code identity and pull request views for one-repository hubs** | [`single-repo-code-identity`](design-single-repo-code-identity.md)<br>[Sketch](decision-single-repo-code-identity.md) | tapestry · solid | T5 | 11h<br>11h | fabric, cwd, gate, git, weaver, drift, webster, lint, loom, landing, pawl, shed, arbiter |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (10)</strong></summary>

Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>1d | board, ide, fabric |

### Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Design the campaign that runs a live-substrate module in a sandbox until it converges, on lym modules as they stand; crucible is a Millhouse-era account only | [`hardener`](design-hardener.md)<br>[Sketch](decision-hardener.md) | 6w<br>10h | burler |

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Run the mutation trial with lym gate mutate**<br>• Replay about twenty landed diffs with lym gate mutate and decide whether to build the mutation report step | [`mutation-trial`](design-mutation-trial.md) | 9m<br>9m |  |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

#### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Some test writes a .git into the shared temp directory and fails unrelated packages**<br>• tmuxkit fails every package running beside the writer; find the test that writes it and fix it | [`stray-tmp-git-writer`](design-stray-tmp-git-writer.md) | 12h<br>12h | gate, tools |
| B2 |  | **A docs darn can land with an empty diff and its report only in the change description**<br>• Land accepts an empty diff silently and dobby reports main tip as the landing; a docs darn should commit its report under docs/research | [`empty-landing-not-flagged`](design-empty-landing-not-flagged.md) | 11h<br>11h | landing, darn |
| B3 |  | **A creel post to SLACK reports ok while Slack is disconnected** | [`creel-slack-post-silent-when-off`](design-creel-slack-post-silent-when-off.md) | 1m<br>1m | creel, slack |

#### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>23h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
