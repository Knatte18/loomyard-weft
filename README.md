# Board

Rendered 2026-11-22 09:50 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (0)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 3d · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 4d · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>18h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric</a> (1)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 3d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Hardener campaign on fabric, one-repository layout**<br>• Hardener run on the one-repository part of fabric only<br>• `hardener: packages ["internal/fabricengine","internal/fabriccli","internal/gitrepo"]`<br>• `loom: notify_parent_on ["Harden-Bouncer","Harden-Burler-Fix"]` | [`harden-fabricengine`](design-harden-fabricengine.md) | hardener | Harden-Bouncer | neve | 14h<br>4m | fabric |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (7)</strong></summary>

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
| T1 | `draft` **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge**<br>• Parent orch settles all halts of a project or beam; operator reached only for a severe halt or a merge-ready beam PR | [`orch-full-autonomy-mode`](design-orch-full-autonomy-mode.md) | motif |  | 1h<br>0m | orch, drift, landing, dobby, batten, pawl, config |

### Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 12d<br>12d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

#### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A stale gate key refuses the orch launch**<br>• claudeengine refused to resume the orch over a retired gate key; no orch until a hand edit | [`orch-launch-refused-by-gate-key`](design-orch-launch-refused-by-gate-key.md) | 7m<br>7m | claude, gate, config, orch |
| B2 |  | **A stale records tracking ref blocks a cancelled slug from running again**<br>• OriginCodeTip reads a deleted records branch through an unpruned tracking ref; dobby refuses the start | [`stale-records-ref-blocks-rerun`](design-stale-records-ref-blocks-rerun.md) | 3m<br>3m | fabric, batten, dobby |

#### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 6d<br>4d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
