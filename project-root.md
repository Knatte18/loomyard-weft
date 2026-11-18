# P0 Root: entries under no project

Rendered 2026-11-18 18:03 UTC; every age below is as of then.
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
| T4 | **Pause holds no shared lock and works on a pair with no reed session**<br>• A held call waits without the fabric push lock, dobby starts no driver for a paused pair, and pause and status skip a pair with no reed session<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pause-after-crash-fixes`](design-pause-after-crash-fixes.md) | darn · solid | Code-Burler-Review | neve | 4h<br>1h | tender, fabric, batten, reed |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 23h<br>22h | board, ide, fabric |

## Notes (12)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 23h<br>23h | shuttle |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Board entry imported from an issue loses the evidence the issue names, and an issue body can arrive empty** | [`import-loses-issue-evidence`](design-import-loses-issue-evidence.md) | 1h<br>1h | board, selfreport |
| E2 |  | **Clean-room rewrite of the most patched modules, landing first**<br>• Rewrite a module and its tests from a written spec alone, interfaces fixed; pilot on landingshed | [`clean-room-module-rewrite`](design-clean-room-module-rewrite.md) | 6h<br>25m | landing |
| E4 |  | **The Plan writer explorer forks cost more than the writer and show no gain**<br>• Drop the Plan writer exploration subagents; they cost about $6.50 per tapestry run | [`plan-writer-explorer-forks`](design-plan-writer-explorer-forks.md) | 5h<br>5h | loom, stencil |

### Features (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F2 |  | **Fabric single-repo mode: the weft lives inside the warp repo, no piggyback repo**<br>• Let a hub run with _lym and .lym as real directories in the warp repo instead of junctions to a separate weft repo, same as today from outside | [`fabric-single-repo-mode`](design-fabric-single-repo-mode.md) | 48m<br>46m | fabric |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>10h | batten, loom, board |
| F4 |  | **Reach note for a diff: the tests reaching each changed function, and changed functions nothing reaches**<br>• A reader of the coverage maps feeds the code review and a read verb for writers; uses 1 and 3 of the map study read through it | [`testmap-reach-note`](design-testmap-reach-note.md) | 4m<br>0m | testkit, gate, burler |
| F5 |  | **Mutation check of the code a diff changes, survivors to the code review**<br>• Mutate only the blocks a diff adds or changes, run only the tests covering each, report surviving mutants; start with a hand trial over about twenty landed diffs | [`testmap-diff-mutation`](design-testmap-diff-mutation.md) | 4m<br>4m | testkit, gate, burler |
| F6 |  | **Failure triage: does a failing test reach what the run changed**<br>• The gate failure report and the fixer brief say which changed functions a failing test reaches, or that it reaches none, so a not-my-fault failure is seen at once | [`testmap-failure-triage`](design-testmap-failure-triage.md) | 4m<br>4m | testkit, gate, darn |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
