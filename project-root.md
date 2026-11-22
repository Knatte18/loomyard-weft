# P0 Root: entries under no project

Rendered 2026-11-22 11:08 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Autonomy mode: the parent orch settles every halt, operator only for severe halts and the beam merge**<br>• Parent orch settles all halts of a project or beam; operator reached only for a severe halt or a merge-ready beam PR | [`orch-full-autonomy-mode`](design-orch-full-autonomy-mode.md) | motif | Discussion-Burler-Review | neve | 2h<br>1h | orch, drift, landing, dobby, batten, pawl, config |
| T5 | **Board README: beam marker, no empty sections, no Notes level**<br>• Declutter the rendered board README in a sampler session | [`readme-declutter`](design-readme-declutter.md) | sampler | Sampler-Session | neve | 1h<br>1h | board |
| T6 | **lym report convergence stays within bounded memory**<br>• convergence grew to 9.5 GB in two minutes over a four-week window and froze the machine.<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`convergence-report-memory`](design-convergence-report-memory.md) | darn · light | Drift-Verify | neve | 20m<br>17m | report |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **A stale records tracking ref blocks a cancelled slug from running again**<br>• OriginCodeTip reads a deleted records branch through an unpruned tracking ref; dobby refuses the start<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`stale-records-ref-blocks-rerun`](design-stale-records-ref-blocks-rerun.md) | darn · light | T2 | 1h<br>1h | fabric, batten, dobby |

## Notes (7)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 12d<br>12d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 4d<br>4d | shuttle |

### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Hardener rounds may run fabric verbs in their own sandbox hubs**<br>• The task-session fabric guard refuses hardener rounds the verbs they must drive on sandbox hubs | [`harden-sandbox-fabric-verbs`](design-harden-sandbox-fabric-verbs.md) | 1h<br>1h | fabric, shed |
| B2 |  | **Sampler session loads the language skills before it edits code**<br>• The sampler spec carries no language skills, so the sampler edits Go without scribe:golang-* and code-quality. | [`sampler-loads-language-skills`](design-sampler-loads-language-skills.md) | 51m<br>51m | loom, shuttle |
| B3 |  | **Hardener live check passes packages go can resolve**<br>• hardenLiveArgs passes bare internal/x paths, which go reads as std packages, so harden-live fails at setup every round. | [`harden-live-bare-package-paths`](design-harden-live-bare-package-paths.md) | 42m<br>42m | gate, loom |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 6d<br>4d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
