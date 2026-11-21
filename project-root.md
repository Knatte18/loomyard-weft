# P0 Root: entries under no project

Rendered 2026-11-21 17:34 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **Webster and plan fix honour parent decisions; full halt reasons**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`webster-parent-calls-flow`](design-webster-parent-calls-flow.md) | darn · solid |  | 5m<br>0m | webster, burler, drift, loom |

## Notes (10)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 12d<br>12d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 3d<br>3d | shuttle |

### Bugs (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Unjudged red-first refusal burns the verify attempts before the parent answers** | [`red-first-unjudged-burns-tries`](design-red-first-unjudged-burns-tries.md) | 2h<br>5m | gate, loom |
| B4 |  | **Burler-fix halts failed when a fork runs git in a scratch repo under .scratch** | [`fork-git-guard-scratch-repo`](design-fork-git-guard-scratch-repo.md) | 5m<br>5m | burler, shuttle |
| B5 |  | **Burler fixer step re-runs on a round an earlier fixer finished, with no reason in the prompt** | [`burler-fixer-rerun-no-reason`](design-burler-fixer-rerun-no-reason.md) | 5m<br>5m | burler |
| B6 |  | **loom-halt note and blocked reason name a gate-findings path that does not exist** | [`halt-names-missing-findings-path`](design-halt-names-missing-findings-path.md) | 5m<br>5m | loom, gate |
| B7 |  | **loom pr-findings fails in PR-Review: NewParentAlign Deps.OpenFabric must not be nil** | [`pr-findings-nil-open-fabric`](design-pr-findings-nil-open-fabric.md) | 5m<br>5m | loom |
| B9 |  | **Stencil build-version refusal at Preflight appends an inapplicable config/model-spec way forward** | [`stencil-version-way-forward`](design-stencil-version-way-forward.md) | 4m<br>4m | stencil, shed |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 6d<br>3d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
