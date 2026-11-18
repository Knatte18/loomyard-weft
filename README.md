# Board

Rendered 2026-11-18 08:08 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-codexengine.md">P1 codexengine</a> (2)</strong></summary>

[`codexengine`](design-codexengine.md) · Age 1d · Owner `neve`

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 11d<br>13h | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 8d<br>12h | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 21h · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>21h | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-test-suite-trim.md">P6 Coverage maps: build them, keep them fresh, and cut the test suite with them</a> (2)</strong></summary>

[`test-suite-trim`](design-test-suite-trim.md) · Age 8d · Owner `neve`

### Tasks (1)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 8d<br>13h | testkit, gate |

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **No lym check compares a map entry with a single-test coverage run**<br>• The trial cross-checked by hand; blocks have no line range and funcs have no profile counterpart | [`map-single-test-crosscheck`](design-map-single-test-crosscheck.md) | 17h<br>25m | gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (18)</strong></summary>

Owner `neve`

### Tasks (5)

Concrete and claimable; only a task can run.

#### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | Discussion-Burler-Fix | neve | 2w<br>19h | claude, shuttle, config |
| T4 | **lym orch launches keep the python guard although deny_python is false**<br>• An orch launch ignores deny_python: false in the hub shuttle config; notice and hook both stay<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`orch-ignores-deny-python`](design-orch-ignores-deny-python.md) | darn · light | Darn-Write | neve | 30m<br>30m | orch, shuttle, config |
| T6 | **Stage the reviewer diff in pieces that fit one Read**<br>• Burler writes the round diff per package under the Read cap, so a reviewer never reads one huge diff<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`review-diff-in-pieces`](design-review-diff-in-pieces.md) | darn · light | Code-Burler-Review | neve | 1h<br>30m | burler, stencil |
| T7 | **A shell guard refusal says nothing ran and names the refused segment**<br>• Saves the agent working out which part of a compound command failed<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`refusal-names-segment`](design-refusal-names-segment.md) | darn · light | Code-Burler-Review | neve | 1h<br>30m | process |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 14h<br>13h | board, ide, fabric |

### Notes (13)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 13h<br>13h | shuttle |

#### Bugs (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **drift judge halt blames the judge for a parent's lym loom decision add during Drift-Verify** | [`drift-judge-parent-decision`](design-drift-judge-parent-decision.md) | 0m<br>0m | drift, loom |
| B2 |  | **Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure** | [`twill-fixer-fork-writes`](design-twill-fixer-fork-writes.md) | 0m<br>0m | burler, lenses |
| B3 |  | **Gate re-prompt fails: shuttle strand not tracked by reed, Plan-Write halts blocked** | [`gate-reprompt-strand-untracked`](design-gate-reprompt-strand-untracked.md) | 0m<br>0m | gate, shuttle, reed |
| B4 |  | **drift: line-range expectation entries never match in diff mode lines** | [`drift-line-range-never-matches`](design-drift-line-range-never-matches.md) | 0m<br>0m | drift |
| B5 |  | **Worktree .scratch is a symlink into .lym/scratch; tools given a path through it see each project under two paths** | [`scratch-symlink-two-paths`](design-scratch-symlink-two-paths.md) | 0m<br>0m | fabric, cwd |
| B6 |  | **shed step re-halts on the shuttle run left running by the earlier interrupted-stop** | [`restep-leftover-shuttle-run`](design-restep-leftover-shuttle-run.md) | 0m<br>0m | shed, shuttle |
| B7 |  | **shuttle start fails with 'session is not idle' when the pane shows an idle input prompt after skill loading** | [`idle-check-after-skill-load`](design-idle-check-after-skill-load.md) | 0m<br>0m | shuttle |

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **Code-Review lets result-level issues through to PR-Review** | [`code-review-result-checks`](design-code-review-result-checks.md) | 0m<br>0m | burler, stencil, drift |

#### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>8m | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
