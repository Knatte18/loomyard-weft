# Board

Rendered 2026-11-01 17:39 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Fewer agent turns, Webster forks first](project-agent-turns.md) (0)

[`agent-turns`](design-agent-turns.md) · Age 3h · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

### Tasks (0)

Concrete and claimable; only a task can run.

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## [P0 Root: entries under no project](project-root.md) (39)

Owner `neve`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **One test binary per package, the tier chosen at run time**<br>• LYM_TEST_TIER selects the tier instead of build tags, taking a cold gate run from 532 test binaries to 182<br>• `profile: solid` | [`gate-one-binary-per-package`](design-gate-one-binary-per-package.md) | darn | Darn-Write | neve | 2h<br>21m | gate, testkit, pattern |
| T5 | **Study: Parts-Write and Parts-Plan-Write against Plan-Write**<br>• Write the plans of two landed runs both ways and compare cost, time, gate pass and one Plan-Review round<br>• `profile: docs` | [`plan-split-bench`](design-plan-split-bench.md) | darn | Darn-Write | neve | 2h<br>2h | loom |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Tune the plan-split writers until a split plan joins with no hand edit**<br>• Tune, replay on two inputs, tune again, at most 3 rounds, starting from the plan-split-bench report | [`plan-split-tune`](design-plan-split-tune.md) | darn | T5<br>T3 | 8m<br>7m | loom, stencil, pattern |

### Notes (36)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 3h<br>46m | tools |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 10d<br>2h | loom, stencil |

#### Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A writer session can end with untracked files in the worktree, which halt Finalize**<br>• A producing session's end gate refuses to end while the worktree holds untracked files outside its outputs | [`writer-leaves-stray-files`](design-writer-leaves-stray-files.md) | 4h<br>3h | shuttle, darn, loom |
| B2 |  | **Dobby reports a healthy run idle in the seconds after its agent session ends**<br>• The no-agent idle clock falls back to the row start in the gap between the session ending and the row writing its history | [`dobby-idle-notice-row-end`](design-dobby-idle-notice-row-end.md) | 5h<br>5h | dobby, shuttle |
| B3 |  | **A batten pass after a landed child reports a false rewind against origin**<br>• Origin equals local, yet batten stops asking which history to keep | [`false-rewind-after-finalize`](design-false-rewind-after-finalize.md) | 1h<br>1h | batten, fabric |
| B4 |  | **lym config --set cannot write a key inside a nested map, such as a gate tier env**<br>• An orch cannot change a gate tier env or command without an interactive editor | [`config-set-nested-keys`](design-config-set-nested-keys.md) | 15m<br>15m | config, gate |
| B5 |  | **Reed loses the Parts-Plan-Write chair strand while its part writers run**<br>• The chair vanished from reed, then the row failed on two missed liveness checks | [`reed-loses-parts-chair`](design-reed-loses-parts-chair.md) | 12m<br>12m | reed, loom |

#### Enhancements (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **The board README shows a task's profile in the Recipe column**<br>• Move the profile from a bullet under the task title into the Recipe column | [`readme-profile-in-recipe-column`](design-readme-profile-in-recipe-column.md) | 5h<br>5h | board |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>1h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 5d<br>5h | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 3d<br>3d | dotnet, gate |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 9h<br>9h | dobby, orch |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 1d<br>10h | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 1d<br>22h | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 1d<br>6h | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 1d<br>5h | fabric |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 2w<br>5d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 2w<br>5d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 10d<br>4d | burler, claude |
| E22 |  | **tokencount counts darn runs and compares recipes**<br>• Count darns with their own segments, so a darn's cost stands beside a tapestry's | [`tokencount-darn-runs`](design-tokencount-darn-runs.md) | 3h<br>3h | tools |
| E24 |  | **A status-bar button asks for a value, so an agent opened from it gets a name**<br>• Button entries take ask, filling {answer}; the agent button asks for the strand name, and term becomes terminal | [`button-ask-name`](design-button-ask-name.md) | 2h<br>2h | reed, config |

#### Features (15)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>5d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 22h<br>2h | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 4h<br>4h | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 22h<br>22h | orch, dobby, arbiter |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 4w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 2d<br>2d | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2d<br>2d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 4h<br>4h | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>55m | board, ide, orch |
| F12 |  | **A darn writer hands per-package batches to Sonnet helpers**<br>• Today lym denies the Agent tool in darn, so a wide change runs every fix through one large Opus context | [`darn-sonnet-helpers`](design-darn-sonnet-helpers.md) | 0m<br>0m | darn, claude, shuttle |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>4d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 10d<br>6h | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>4d | burler, shed, orch, quarry, report |
