# P0 Root: entries under no project

Rendered 2026-11-19 19:42 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Raddle checks pass a part slug that Raddle-File then refuses as too long**<br>• The proposal checks do not test the board slug length limit, so an approved split stops at Raddle-File<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`raddle-slug-length-check`](design-raddle-slug-length-check.md) | darn · light | Darn-Write | neve | 1h<br>8m | shed |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Gate fixes: verdict from parsed results, only declared gate keys in prompts, env override warning**<br>• Three gate faults seen in .NET hubs: a console logger flips the verdict, prompts name an undeclared gate key, and tier env overrides go unreported<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`gate-dotnet-fixes`](design-gate-dotnet-fixes.md) | darn · solid |  | 2m<br>0m | gate, dotnet, stencil, darn |

## Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 10d<br>10d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 2d<br>2d | shuttle |

### Bugs (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **hardenshed reaper misses orphans whose cwd is under a _lym link** | [`harden-reap-lym-link-cwd`](design-harden-reap-lym-link-cwd.md) | 2m<br>2m | shed, process |
| B4 |  | **Finalize halts on a pull request whose mergeability GitHub has not computed yet** | [`finalize-mergeable-unknown`](design-finalize-mergeable-unknown.md) | 1m<br>1m | landing |
| B5 |  | **Verify triage lists verify-command as a flaky identity with no failure output** | [`verify-triage-command-identity`](design-verify-triage-command-identity.md) | 1m<br>1m | webster, gate |
| B6 |  | **Flaky verify failure in internal/reedcli on the first integration-suite run** | [`reedcli-flaky-verify`](design-reedcli-flaky-verify.md) | 1m<br>1m | reed |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Burler fix round: per-finding commit rule conflicts with shared files; review location misses the seam** | [`burler-fix-shared-file-commits`](design-burler-fix-shared-file-commits.md) | 1m<br>1m | burler, stencil |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 4d<br>1d | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>2w | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
