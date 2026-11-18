# Board

Rendered 2026-11-18 09:44 UTC; every age below is as of then.
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
| R1 |  | **Codex as assistant Code-Review reviewer on real darn runs**<br>• Set Codex as the non-gating assistant reviewer on real runs and read what it adds beside the Claude reviewer | [`codex-assist-review-trial`](design-codex-assist-review-trial.md) | 11d<br>42m | codex, shuttle |
| R2 |  | **Codex support in levels: review, darn, then the rest of tapestry**<br>• Say lym supports Codex only at a level proven on real runs; the first work is a darn-writer bench | [`codex-support-levels`](design-codex-support-levels.md) | 8d<br>42m | codex, shuttle |

</details>

<details open>
<summary><strong><a href="project-borrowed-orchestrator-ideas.md">P2 Ideas borrowed from comparable agent orchestrators</a> (1)</strong></summary>

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 22h · Owner `neve`

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
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>22h | burler, shed, orch, quarry, report |

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
| T2 | `draft` **Cut the existing redundant tests of every package, found by the coverage maps** | [`redundancy-sweep-existing`](design-redundancy-sweep-existing.md) | darn · solid |  | 8d<br>14h | testkit, gate |

### Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **No lym check compares a map entry with a single-test coverage run**<br>• The trial cross-checked by hand; blocks have no line range and funcs have no profile counterpart | [`map-single-test-crosscheck`](design-map-single-test-crosscheck.md) | 18h<br>2h | gate |

</details>

<details open>
<summary><strong><a href="project-root.md">P0 Root: entries under no project</a> (15)</strong></summary>

Owner `neve`

### Tasks (7)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Drift: a parent decision trips the judge snapshot check, and line-range expectations never match**<br>• Two drift bugs: parent decision commits blamed on the judge, and line-range entries that never match<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drift-judge-and-line-range`](design-drift-judge-and-line-range.md) | darn · solid | Describe | neve | 1h<br>48m | drift, loom |
| T5 | **Shuttle start: idle check misses an idle prompt, and a review strand misses its prompt turn**<br>• Two seat start bugs in shuttle prompt delivery<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`shuttle-start-idle-and-prompt`](design-shuttle-start-idle-and-prompt.md) | darn · solid | Code-Burler-Fix | neve | 1h<br>48m | shuttle, burler |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Leftover strands and shuttle runs halt a gate re-prompt and a re-step**<br>• Gate re-prompt to an untracked strand, and a re-step tripping on a leftover shuttle run<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`leftover-strands-and-runs`](design-leftover-strands-and-runs.md) | darn · solid | T5 | 1h<br>48m | gate, shuttle, reed, shed |
| T8 | **Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure**<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`twill-fixer-fork-writes`](design-twill-fixer-fork-writes.md) | darn · light | T5 | 1h<br>48m | burler, lenses |

#### Layer B (1)

Starts when every entry it names under After is done.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **C# hub: silent dotnet test exit, and runs failing in a hub with no tier0 or only inert changes**<br>• Make a refused dotnet test say so; let a hub with no tier0 or an all-inert change run and land<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`csharp-hub-gate-fixes`](design-csharp-hub-gate-fixes.md) | darn · solid | T6 | 1h<br>48m | dotnet, process, gate, config |

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **Board viewer rendered beside the README, opened by lym board open and a VS Code key** | [`board-viewer-build`](design-board-viewer-build.md)<br>[Sketch](decision-board-viewer-build.md) | sampler · solid |  | 15h<br>14h | board, ide, fabric |
| T9 | `draft` **Claude accounts per worktree: pick, pins, headroom, transcript carry and switch on a usage limit** | [`claude-accounts-per-worktree`](design-claude-accounts-per-worktree.md)<br>[Sketch](decision-claude-accounts-per-worktree.md) | tapestry · solid | T6 | 6m<br>6m | claude, shuttle, reed, orch, batten, cwd, config, tools |

### Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Code-Review lets result-level issues through to PR-Review** | [`code-review-result-checks`](design-code-review-result-checks.md)<br>[Sketch](decision-code-review-result-checks.md) | 1h<br>7m | burler, stencil, drift |

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 8d<br>8d | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 14h<br>14h | shuttle |

#### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **selfreport -b takes a file path as the body text, losing the report**<br>• An agent passed a file path to -b and the issue body became the path | [`selfreport-body-path-as-text`](design-selfreport-body-path-as-text.md) | 48m<br>48m | selfreport |
| B2 |  | **Seed-IDE fails: records push calls main-weft ended on origin**<br>• A new run fails at Seed-IDE because the prime records branch is read as ended on origin; a rerun passes | [`records-push-main-weft-ended`](design-records-push-main-weft-ended.md) | 30m<br>30m | fabric, batten |

#### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>3w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 2d<br>1h | batten, loom, board |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>12d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

</details>
