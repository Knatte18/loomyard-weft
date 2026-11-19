# Board

Rendered 2026-11-19 18:28 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (2)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 1d · Owner `neve`

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
| T1 | `draft` **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | darn · solid |  | 1d<br>6h | fabric, loom, board |
| T11 | `draft` **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | darn · solid | T1 | 22h<br>7h | docs |

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>1h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-testmap-uses.md">P3 Put the per-test coverage maps to use</a> (1)</strong></summary>

[`testmap-uses`](design-testmap-uses.md) · Age 1d · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **Measure whether the reach and mutation reports change what runs land**<br>• Over 10 landed darn runs, count how often the reach note, reach turn and mutation survivors led to added tests or fixes<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`testmap-use-measure`](design-testmap-use-measure.md) | darn · docs |  | 5h<br>5h | testmap, gate |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records on each code branch, the board on lym-bolt</a> (1)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 22h · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T7 | **One-repository hubs redesigned: records on their own branches, the layout known only to fabric**<br>• Redesign single-repo mode so records live on their own branches in the code repository, wired as _lym like weft, and decide what of f06acf39 stays | [`single-repo-records-branches`](design-single-repo-records-branches.md)<br>[Sketch](decision-single-repo-records-branches.md) | raddle | Raddle-File | neve | 6h<br>29m | fabric |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (9)</strong></summary>

Owner `neve`

### Tasks (4)

Concrete and claimable; only a task can run.

#### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **A shell -c script that removes files stalls a run on a permission dialog**<br>• The shell guard does not look inside a shell -c script, so Claude Code asks for permission and the run stalls until someone answers<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`shellguard-sh-c-rm-prompt`](design-shellguard-sh-c-rm-prompt.md) | darn · light | Code-Burler-Review | neve | 1h<br>29m | process |
| T3 | **Board viewer page with live editing through a loopback board server, opened by lym board open and a VS Code key** | [`board-viewer-server`](design-board-viewer-server.md)<br>[Sketch](decision-board-viewer-server.md) | tapestry · solid | Parent-Align | neve | 1h<br>1h | board, ide, fabric, process, cwd |
| T4 | **Bug runs prove their test was red before the fix, and name the root cause in a trailer**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`bug-red-first-gate`](design-bug-red-first-gate.md) | darn · light | Darn-Write | neve | 5h<br>3h | gate, testmap |
| T6 | **A hardenshed test leaves .lock in the package directory**<br>• A record write in a hardenshed test takes its lock beside a relative or empty path, so internal/hardenshed/.lock appears in the worktree and blocks Finalize<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hardenshed-test-stray-lock`](design-hardenshed-test-stray-lock.md) | darn · light | Code-Burler-Fix | neve | 46m<br>29m | gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (5)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 10d<br>10d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 1d<br>1d | shuttle |

#### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Raddle checks pass a part slug that Raddle-File then refuses as too long**<br>• The proposal checks do not test the board slug length limit, so an approved split stops at Raddle-File | [`raddle-slug-length-check`](design-raddle-slug-length-check.md) | 0m<br>0m | shed |

#### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 4d<br>1d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
