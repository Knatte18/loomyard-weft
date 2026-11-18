# P0 Root: entries under no project

Rendered 2026-11-18 09:00 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (8)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Drift: a parent decision trips the judge snapshot check, and line-range expectations never match**<br>• Two drift bugs: parent decision commits blamed on the judge, and line-range entries that never match<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drift-judge-and-line-range`](design-drift-judge-and-line-range.md) | darn · solid | Darn-Write | neve | 53m<br>4m | drift, loom |
| T9 | **Agents must never build or check out a repository under .scratch, .lym or _lym**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`scratch-symlink-two-paths`](design-scratch-symlink-two-paths.md) | darn · light | Drift-Expect | neve | 52m<br>4m | stencil |
| T10 | **Code-Review lets result-level issues through to PR-Review** | [`code-review-result-checks`](design-code-review-result-checks.md) | motif | Discussion-Write | neve | 52m<br>4m | burler, stencil, drift |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **Shuttle start: idle check misses an idle prompt, and a review strand misses its prompt turn**<br>• Two seat start bugs in shuttle prompt delivery<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`shuttle-start-idle-and-prompt`](design-shuttle-start-idle-and-prompt.md) | darn · solid |  | 52m<br>4m | shuttle, burler |

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Leftover strands and shuttle runs halt a gate re-prompt and a re-step**<br>• Gate re-prompt to an untracked strand, and a re-step tripping on a leftover shuttle run<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`leftover-strands-and-runs`](design-leftover-strands-and-runs.md) | darn · solid | T5 | 52m<br>4m | gate, shuttle, reed, shed |
| T8 | **Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`twill-fixer-fork-writes`](design-twill-fixer-fork-writes.md) | darn · light | T5 | 52m<br>4m | burler, lenses |

### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **C# hub: silent dotnet test exit, and runs failing in a hub with no tier0 or only inert changes**<br>• Make a refused dotnet test say so; let a hub with no tier0 or an all-inert change run and land<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`csharp-hub-gate-fixes`](design-csharp-hub-gate-fixes.md) | darn · solid | T6 | 52m<br>4m | dotnet, process, gate, config |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 14h<br>13h | board, ide, fabric |

## Notes (7)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md)<br>[Sketch](decision-claude-account-pick-simple.md) | 2w<br>41m | claude, shuttle, config |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 14h<br>14h | shuttle |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **selfreport -b takes a file path as the body text, losing the report**<br>• An agent passed a file path to -b and the issue body became the path | [`selfreport-body-path-as-text`](design-selfreport-body-path-as-text.md) | 4m<br>4m | selfreport |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>1h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T4 | **lym orch launches keep the python guard although deny_python is false**<br>• An orch launch ignores deny_python: false in the hub shuttle config; notice and hook both stay<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-ignores-deny-python`](design-orch-ignores-deny-python.md) | 1h<br>1h | orch, shuttle, config |
