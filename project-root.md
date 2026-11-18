# P0 Root: entries under no project

Rendered 2026-11-18 08:31 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **lym orch launches keep the python guard although deny_python is false**<br>• An orch launch ignores deny_python: false in the hub shuttle config; notice and hook both stay<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-ignores-deny-python`](design-orch-ignores-deny-python.md) | darn · light | Code-Bouncer | neve | 54m<br>54m | orch, shuttle, config |
| T6 | **Stage the reviewer diff in pieces that fit one Read**<br>• Burler writes the round diff per package under the Read cap, so a reviewer never reads one huge diff<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`review-diff-in-pieces`](design-review-diff-in-pieces.md) | darn · light | landing · Finalize | neve | 1h<br>53m | burler, stencil |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 14h<br>13h | board, ide, fabric |

## Notes (17)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md)<br>[Sketch](decision-claude-account-pick-simple.md) | 2w<br>12m | claude, shuttle, config |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 13h<br>13h | shuttle |

### Bugs (10)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **drift judge halt blames the judge for a parent's lym loom decision add during Drift-Verify** | [`drift-judge-parent-decision`](design-drift-judge-parent-decision.md) | 23m<br>23m | drift, loom |
| B2 |  | **Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure** | [`twill-fixer-fork-writes`](design-twill-fixer-fork-writes.md) | 23m<br>23m | burler, lenses |
| B3 |  | **Gate re-prompt fails: shuttle strand not tracked by reed, Plan-Write halts blocked** | [`gate-reprompt-strand-untracked`](design-gate-reprompt-strand-untracked.md) | 23m<br>23m | gate, shuttle, reed |
| B4 |  | **drift: line-range expectation entries never match in diff mode lines** | [`drift-line-range-never-matches`](design-drift-line-range-never-matches.md) | 23m<br>23m | drift |
| B5 |  | **Agents must never build or check out a repository under .scratch, .lym or _lym** | [`scratch-symlink-two-paths`](design-scratch-symlink-two-paths.md) | 23m<br>5m | stencil |
| B6 |  | **shed step re-halts on the shuttle run left running by the earlier interrupted-stop** | [`restep-leftover-shuttle-run`](design-restep-leftover-shuttle-run.md) | 23m<br>23m | shed, shuttle |
| B7 |  | **shuttle start fails with 'session is not idle' when the pane shows an idle input prompt after skill loading** | [`idle-check-after-skill-load`](design-idle-check-after-skill-load.md) | 23m<br>23m | shuttle |
| B8 |  | **Review strand starts without the turn naming its prompt file, also after restart** | [`review-strand-no-prompt-turn`](design-review-strand-no-prompt-turn.md) | 23m<br>23m | shuttle, burler |
| B9 |  | **dotnet test exits 0 with no output in a darn session while lym gate test is off limits** | [`dotnet-test-silent-exit`](design-dotnet-test-silent-exit.md) | 23m<br>23m | dotnet, process |
| B10 |  | **A run in a hub with no declared tier0 fails at shed build, even a docs-only darn** | [`no-tier0-fails-shed-build`](design-no-tier0-fails-shed-build.md) | 23m<br>23m | gate, config |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **Code-Review lets result-level issues through to PR-Review** | [`code-review-result-checks`](design-code-review-result-checks.md) | 23m<br>23m | burler, stencil, drift |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>31m | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
