# P0 Root: entries under no project

Rendered 2026-11-18 10:09 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Code-Review lets result-level issues through to PR-Review** | [`code-review-result-checks`](design-code-review-result-checks.md)<br>[Sketch](decision-code-review-result-checks.md) | raddle | Raddle-Write | neve | 2h<br>14m | burler, stencil, drift |
| T6 | **Leftover strands and shuttle runs halt a gate re-prompt and a re-step**<br>• Gate re-prompt to an untracked strand, and a re-step tripping on a leftover shuttle run<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`leftover-strands-and-runs`](design-leftover-strands-and-runs.md) | darn · solid | Preflight | neve | 2h<br>1h | gate, shuttle, reed, shed |
| T8 | **Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`twill-fixer-fork-writes`](design-twill-fixer-fork-writes.md) | darn · light | starting · Seed-Child | neve | 2h<br>1h | burler, lenses |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **C# hub: silent dotnet test exit, and runs failing in a hub with no tier0 or only inert changes**<br>• Make a refused dotnet test say so; let a hub with no tier0 or an all-inert change run and land<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`csharp-hub-gate-fixes`](design-csharp-hub-gate-fixes.md) | darn · solid | T6 | 2h<br>1h | dotnet, process, gate, config |
| T9 | **Claude accounts per worktree: pick, pins, headroom, transcript carry and switch on a usage limit** | [`claude-accounts-per-worktree`](design-claude-accounts-per-worktree.md)<br>[Sketch](decision-claude-accounts-per-worktree.md) | tapestry · solid | T6 | 30m<br>30m | claude, shuttle, reed, orch, batten, cwd, config, tools |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 16h<br>15h | board, ide, fabric |

## Notes (7)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 15h<br>15h | shuttle |

### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **selfreport -b takes a file path as the body text, losing the report**<br>• An agent passed a file path to -b and the issue body became the path | [`selfreport-body-path-as-text`](design-selfreport-body-path-as-text.md) | 1h<br>1h | selfreport |
| B2 |  | **Seed-IDE fails: records push calls main-weft ended on origin**<br>• A new run fails at Seed-IDE because the prime records branch is read as ended on origin; a rerun passes | [`records-push-main-weft-ended`](design-records-push-main-weft-ended.md) | 55m<br>55m | fabric, batten |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>2h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T5 | **Shuttle start: idle check misses an idle prompt, and a review strand misses its prompt turn**<br>• Two seat start bugs in shuttle prompt delivery<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`shuttle-start-idle-and-prompt`](design-shuttle-start-idle-and-prompt.md) | 2h<br>1h | shuttle, burler |
