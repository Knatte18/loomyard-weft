# Board

Rendered 2026-11-21 17:29 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (2)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 3d · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

### Tasks (2)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | darn · solid | Code-Burler-Fix | neve | 3d<br>1d | fabric, loom, board |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | darn · solid | T1 | 2d<br>2d | docs |

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>1h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-single-repo-hub.md">P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric</a> (3)</strong></summary>

[`single-repo-hub`](design-single-repo-hub.md) · Age 2d · Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **One-repository sandbox: create the repo and clone the hub**<br>• Create a private sandbox repo with gh, clone it as a one-repository hub, check refs on origin<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-setup`](design-onerepo-sandbox-setup.md) | darn · docs |  | 0m<br>0m | fabric |
| T3 | `draft` **One-repository sandbox: run tasks to landing**<br>• Run sequential, concurrent and abandoned tasks in the sandbox hub and check origin after each<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-tasks`](design-onerepo-sandbox-tasks.md) | darn · docs | T2 | 0m<br>0m | fabric, landing |
| T4 | `draft` **One-repository sandbox: second clone and namespace hand check**<br>• Clone the sandbox again, land a task from it, and grep fabric and gitrepo for un-namespaced records refs<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-reclone`](design-onerepo-sandbox-reclone.md) | darn · docs | T3 | 0m<br>0m | fabric, git |

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (9)</strong></summary>

Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 12d<br>12d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 3d<br>3d | shuttle |

#### Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Unjudged red-first refusal burns the verify attempts before the parent answers** | [`red-first-unjudged-burns-tries`](design-red-first-unjudged-burns-tries.md) | 1h<br>0m | gate, loom |
| B2 |  | **Plan amendment to an unchanged card is approved without being carried into it** | [`plan-amendment-not-carried`](design-plan-amendment-not-carried.md) | 0m<br>0m | burler, webster, loom |
| B3 |  | **webster: halt note truncates drift gate message; no way to accept expected drift** | [`webster-drift-halt-truncated`](design-webster-drift-halt-truncated.md) | 0m<br>0m | webster, drift |
| B4 |  | **Burler-fix halts failed when a fork runs git in a scratch repo under .scratch** | [`fork-git-guard-scratch-repo`](design-fork-git-guard-scratch-repo.md) | 0m<br>0m | burler, shuttle |
| B5 |  | **Burler fixer step re-runs on a round an earlier fixer finished, with no reason in the prompt** | [`burler-fixer-rerun-no-reason`](design-burler-fixer-rerun-no-reason.md) | 0m<br>0m | burler |

#### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 6d<br>3d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
