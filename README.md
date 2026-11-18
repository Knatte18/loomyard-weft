# Board

Rendered 2026-11-18 18:12 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

<details open>
<summary><strong><a href="project-code-cleanup.md">P1 Code cleanup: cut and simplify lym code and tests</a> (4)</strong></summary>

[`code-cleanup`](design-code-cleanup.md) · Age 48m · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (4)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **Comments: check what the big comment rewrite left, before any new pass**<br>• Last week a run of nearly 3,000 agents rewrote docstrings and doc files; measure what is still long before deciding on more | [`comment-trim-second-pass`](design-comment-trim-second-pass.md) | 48m<br>48m | docs |
| E5 |  | **Design: simplify interfaces that make every caller handle too much**<br>• For example one result type instead of a four-value return handled at 60 call sites; find such spots per module and fix them one module at a time | [`design-simplify-callers`](design-design-simplify-callers.md) | 48m<br>48m | fabric, landing |
| E6 |  | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | 48m<br>48m | fabric, loom, board |
| E7 |  | **Remove features nobody uses**<br>• Find unreachable code mechanically and list features with no recent use; the operator decides which go | [`drop-unused-features`](design-drop-unused-features.md) | 48m<br>48m | tools |

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T1 | **Find real uses for the per-test coverage maps**<br>• A research run on Fable searches the web and the code for what per-test coverage maps can do for lym and its agents, and ranks the uses<br>• `darn: writer fable[high]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`testmap-use-cases`](design-testmap-use-cases.md) | 43m<br>7m | testmap, gate |

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 1d · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>1d | burler, shed, orch, quarry, report |

</details>

<details open>
<summary><strong><a href="project-testmap-uses.md">P3 Put the per-test coverage maps to use</a> (1)</strong></summary>

[`testmap-uses`](design-testmap-uses.md) · Age 7m · Owner `neve`

The reach note, the diff mutation check and failure triage, built on one reader of the maps

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Coverage map uses: reach note, failure triage and diff mutation check**<br>• One reader of the maps feeding a reach note for writer and review, failure triage in the gate report and fixer brief, and a mutation check of a diff changed blocks | [`testmap-uses-design`](design-testmap-uses-design.md) | motif | Discussion-Write | neve | 13m<br>4m | testmap, gate, burler, darn |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (11)</strong></summary>

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **Pause holds no shared lock and works on a pair with no reed session**<br>• A held call waits without the fabric push lock, dobby starts no driver for a paused pair, and pause and status skip a pair with no reed session<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`pause-after-crash-fixes`](design-pause-after-crash-fixes.md) | darn · solid | landing · Finalize | neve | 4h<br>1h | tender, fabric, batten, reed |
| T5 | **Bench the explorer subagents of the Discussion and Plan writers: with against without**<br>• Same frozen inputs, the writers run with and without the Agent-tool explore and lookup subagents; compare cost, time and plan or record quality<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`plan-writer-explorer-forks`](design-plan-writer-explorer-forks.md) | darn · docs | Darn-Write | neve | 5h<br>1m | loom, stencil |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 1d<br>23h | board, ide, fabric |

### Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 9d<br>9d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 23h<br>23h | shuttle |

#### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Board entry imported from an issue loses the evidence the issue names, and an issue body can arrive empty** | [`import-loses-issue-evidence`](design-import-loses-issue-evidence.md) | 1h<br>1h | board, selfreport |
| E2 |  | **Clean-room rewrite of the most patched modules, landing first**<br>• Rewrite a module and its tests from a written spec alone, interfaces fixed; pilot on landingshed | [`clean-room-module-rewrite`](design-clean-room-module-rewrite.md) | 6h<br>34m | landing |

#### Features (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F2 |  | **Fabric single-repo mode: the weft lives inside the warp repo, no piggyback repo**<br>• Let a hub run with _lym and .lym as real directories in the warp repo instead of junctions to a separate weft repo, same as today from outside | [`fabric-single-repo-mode`](design-fabric-single-repo-mode.md) | 57m<br>54m | fabric |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 3d<br>10h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>13d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
