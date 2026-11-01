# Board

Rendered 2026-11-01 15:04 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Prime

The root project: the entries under no other project.

### [P1 Orch instructions in two layers: lym's role and the repository's rules in heddle](project-orch-instructions.md) (3)

`orch-instructions` · Age 1h · Owner `neve`

One fixed orch role for every repository, repository orch rules in heddle through lym orch rules, and the orch note for run state only

#### Tasks (3)

Concrete and claimable; only a task can run.

##### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **lym's design invariants as PATTERN entries**<br>• doc.go as documentation only, fabric's isolation, no hard-coded names and the other code invariants become PATTERN entries | [`pattern-design-principles`](design-pattern-design-principles.md) | darn | Finalize | neve | 1h<br>6m | pattern |
| T10 | **Heddle gains an orch-prompt target, and lym orch rules stores in heddle**<br>• A heddle target for orch sessions only replaces the hub-wide and local orch rules files<br>• `profile: solid` | [`heddle-orch-target`](design-heddle-orch-target.md) | darn | Darn-Write | neve | 1h<br>6m | heddle, orch |

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **One fixed orch role for every repository, brought up to date**<br>• The rules for every orch move into the shipped role, which settles run-level calls itself and has no board copy | [`orch-role-current`](design-orch-role-current.md) | darn | T10 | 1h<br>6m | orch, stencil |

#### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

### [P2 Fewer agent turns, Webster forks first](project-agent-turns.md) (9)

`agent-turns` · Age 48m · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

#### Tasks (1)

Concrete and claimable; only a task can run.

##### Ready (0)

Waits on nothing open; can start now.

_None._

##### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **The gate reports every failure at once, lint included, in one readable result**<br>• lym gate test prints a compact list of every build, test and lint failure with file:line and values, and the log path | [`gate-failure-report`](design-gate-failure-report.md) | darn | T6 | 48m<br>37m | gate, lint |

#### Notes (8)

Not tasks: ideas and observations, merged into a task when one is promoted.

##### Enhancements (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E13 |  | **Trial: agents settle every edit before the first, then gate once**<br>• An imperative heddle instruction: read, decide all edits, make them, gate once and fix the whole report in one round | [`plan-edits-before-gate`](design-plan-edits-before-gate.md) | 18m<br>18m | heddle, stencil, claude |
| E14 |  | **A reviewer or fixer wakes once for all its forks, not once per fork**<br>• 79 no-call wake-up turns at full context over three runs, 8-9% of the forking review roles weight | [`fork-results-one-wake`](design-fork-results-one-wake.md) | 13m<br>13m | burler, stencil |
| E15 |  | **A card that retires a name sweeps the whole tree, tests included, first**<br>• The implementer greps every retired name across the tree and test data before its first edit, and reads and edits all hits in one message each | [`retired-name-sweep`](design-retired-name-sweep.md) | 48m<br>48m | webster, stencil |
| E17 |  | **Agents read a file whole or all its spans in one message, not in slices**<br>• No reading one file in three to five slices over as many turns, nor re-reading it | [`read-whole-not-slices`](design-read-whole-not-slices.md) | 48m<br>48m | webster, stencil |
| E18 |  | **Agents still try the forms lym's guards refuse, at a turn each**<br>• Python and raw go build or go vet attempts are refused and retried; find why the steering does not hold and close it | [`hook-refused-turns`](design-hook-refused-turns.md) | 46m<br>42m | process, claude |
| E21 |  | **Trial: the batching instruction in the rendered CLAUDE.local.md through heddle**<br>• Put the batching instruction into heddle's claude-file target and measure calls per turn with forkaudit against today's 1.7 | [`batching-in-claude-local`](design-batching-in-claude-local.md) | 46m<br>46m | claude, heddle |
| E23 |  | **The round-1 Plan reviewer gathers in one call a turn**<br>• 1.05 calls a turn and 41% of its weight removable by merging, though it carries the batching sentence | [`plan-reviewer-batching`](design-plan-reviewer-batching.md) | 13m<br>13m | burler, stencil |

##### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F12 |  | **A card gate failure goes to a cold, cheap fixer instead of the fork**<br>• Go spawns a fresh fixer with the gate report, the card and its diff, so the fork's large context does not run the fix loop | [`card-gate-cold-fixer`](design-card-gate-cold-fixer.md) | 48m<br>46m | webster, gate |

### Tasks (5)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **A long gate run answers within the cache lifetime, so the waiting agent keeps its prompt cache warm**<br>• lym gate test caps the run wait like the slot wait, with the cap a config value beside the engine cache lifetime<br>• `profile: light` | [`gate-run-keeps-cache-warm`](design-gate-run-keeps-cache-warm.md) | tapestry | Code-Burler-Fix | neve | 20h<br>3h | gate, shuttle, config |
| T7 | **One verb to take work over on another machine, leaving every run in a known state**<br>• Moving to another machine is many hand steps, and afterwards run states are unclear | [`machine-switch`](design-machine-switch.md) | tapestry | Code-Burler-Fix | neve | 1d<br>3h | fabric, dobby, batten |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **A concurrent fetch of the same ref fails a batten pass and spawns a tackler**<br>• A fetch into tracking refs waits under the push lock, as fabric's writes wait under theirs<br>• `profile: solid` | [`reconcile-fetch-race`](design-reconcile-fetch-race.md) | darn | T7 | 2h<br>17m | batten, fabric |

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | `draft` **One test binary per package, the tier chosen at run time**<br>• LYM_TEST_TIER selects the tier instead of build tags, taking a cold gate run from 532 test binaries to 182 | [`gate-one-binary-per-package`](design-gate-one-binary-per-package.md) | tapestry |  | 13m<br>13m | gate, testkit, pattern |
| T5 | `draft` **Study: Parts-Write and Parts-Plan-Write against Plan-Write**<br>• Write the plans of two landed runs both ways and compare cost, time, gate pass and one Plan-Review round<br>• `profile: docs` | [`plan-split-bench`](design-plan-split-bench.md) | darn |  | 13m<br>13m | loom |

### Notes (31)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 9d<br>16m | loom, stencil |

#### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A writer session can end with untracked files in the worktree, which halt Finalize**<br>• A producing session's end gate refuses to end while the worktree holds untracked files outside its outputs | [`writer-leaves-stray-files`](design-writer-leaves-stray-files.md) | 2h<br>39m | shuttle, darn, loom |
| B2 |  | **Dobby reports a healthy run idle in the seconds after its agent session ends**<br>• The no-agent idle clock falls back to the row start in the gap between the session ending and the row writing its history | [`dobby-idle-notice-row-end`](design-dobby-idle-notice-row-end.md) | 2h<br>2h | dobby, shuttle |

#### Enhancements (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **The board README shows a task's profile in the Recipe column**<br>• Move the profile from a bullet under the task title into the Recipe column | [`readme-profile-in-recipe-column`](design-readme-profile-in-recipe-column.md) | 3h<br>3h | board |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>4h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 4d<br>3h | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 3d<br>3d | dotnet, gate |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 7h<br>7h | dobby, orch |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 1d<br>8h | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 1d<br>20h | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 1d<br>4h | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 22h<br>3h | fabric |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 2w<br>5d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 13d<br>5d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 9d<br>4d | burler, claude |
| E22 |  | **tokencount counts darn runs and compares recipes**<br>• Count darns with their own segments, so a darn's cost stands beside a tapestry's | [`tokencount-darn-runs`](design-tokencount-darn-runs.md) | 46m<br>46m | tools |
| E24 |  | **A status-bar button asks for a value, so an agent opened from it gets a name**<br>• Button entries take ask, filling {answer}; the agent button asks for the strand name, and term becomes terminal | [`button-ask-name`](design-button-ask-name.md) | 11m<br>11m | reed, config |

#### Features (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>5d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 19h<br>16m | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 2h<br>2h | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 19h<br>19h | orch, dobby, arbiter |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 4w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 2d<br>2d | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2d<br>2d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 2h<br>2h | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>3h | board, ide, orch |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>4d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 9d<br>3h | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>3d | burler, shed, orch, quarry, report |
