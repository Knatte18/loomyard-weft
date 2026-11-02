# Board

Rendered 2026-11-02 16:03 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Fewer agent turns, fewer tokens](project-turn-savings.md) (4)

`turn-savings` · Age 21h · Owner `osl-1046`

Token savings by cutting agent turns, above all late turns at large context

### Tasks (1)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **One cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Darn-Write, plan, discussion or Webster card gate fails, Go spawns one shared cold fixer with the gate report, generalised from webster fix-verify | [`cold-gate-fixer`](design-cold-gate-fixer.md) | tapestry · light | Webster | osl-1046 | 21h<br>21h | webster, loom, darn, gate |

#### Ready (0)

Waits on nothing open; can start now.

_None._

### Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 1d<br>21h | tools |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Bench the guidance that sends Plan and Discussion writers' lookups to parallel Sonnet subagents against a writer given none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 21h<br>2h | loom |
| R3 |  | **Measure the effect of the batching breach hook**<br>• After the batching breach hook lands, compare breach turns and context tokens against the 28-session baseline and tune its thresholds | [`batching-hook-effect`](design-batching-hook-effect.md) | 4h<br>2h | claude, shuttle, tools |

## [P0 Root: entries under no project](project-root.md) (36)

Owner `osl-1046`

### Tasks (3)

Concrete and claimable; only a task can run.

#### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **fabricengine tier1 hits the 10m go test timeout under load**<br>• internal/fabricengine tier1 panics with test timed out after 10m when the machine load is about 30 | [`fabricengine-tier1-timeout-load`](design-fabricengine-tier1-timeout-load.md) | darn · light | Finalize | osl-1046 | 1h<br>1h | gate, fabric |

#### Ready (0)

Waits on nothing open; can start now.

_None._

#### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Tune the plan-split writers until a split plan joins with no hand edit**<br>• Tune, replay on two inputs, tune again, at most 3 rounds, starting from the plan-split-bench report | [`plan-split-tune`](design-plan-split-tune.md) | darn |  | 22h<br>21h | loom, stencil, pattern |
| T8 | `draft` **The landing merge fixer loads no prose, code-quality or testing skill**<br>• FixerTable sets the fixer seat's skills to the language skills alone, unlike every other code-writing seat | [`landing-fixer-base-skills`](design-landing-fixer-base-skills.md) | darn · light |  | 3h<br>3h | landing, shuttle |

### Notes (33)

Not tasks: ideas and observations, merged into a task when one is promoted.

#### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 10d<br>3h | loom, stencil |

#### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **The sampler agent gets no PATTERN directive**<br>• The sampler writes and commits code but its prompt carries no PATTERN directive, unlike every other code-writing role | [`sampler-lacks-pattern-directive`](design-sampler-lacks-pattern-directive.md) | 2m<br>2m | loom, stencil, pattern |
| B2 |  | **Beam orch cannot set a board entry's profile: refused as a task session** | [`beam-orch-cannot-set-profile`](design-beam-orch-cannot-set-profile.md) | 4h<br>4h | board, orch, workspaces |
| B3 |  | **gitexec TestRun_RemoteBounds fails under machine load**<br>• The 200 ms stall bound of the never-answering remote test kills git before its ext script writes its count file when the machine is loaded | [`gitexec-remote-bounds-load-flaky`](design-gitexec-remote-bounds-load-flaky.md) | 1h<br>1h | git, gate |

#### Enhancements (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Finalize releases the parent lock while its fixer works**<br>• A red verify in Finalize frees the parent lock before the fixer starts, then requeues, merges and verifies again, so green runs land while one run fixes | [`finalize-fix-outside-lock`](design-finalize-fix-outside-lock.md) | 20h<br>20h | landing, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>23h | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 6d<br>1d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 4d<br>4d | dotnet, gate |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 1d<br>1d | dobby, orch |
| E6 |  | **A task with no recipe runs darn, not tapestry**<br>• The default recipe becomes darn, set in board.yaml, so a small task promoted without a recipe does not start a full tapestry run | [`default-recipe-darn`](design-default-recipe-darn.md) | 21h<br>21h | board, batten |
| E8 |  | **The orch role explains how to make a project into a beam**<br>• The prime orch in the Models hub did not know beam create is the only way, and patched a plain task run by hand. | [`orch-role-making-a-beam`](design-orch-role-making-a-beam.md) | 5h<br>5h | orch, pawl |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 2d<br>1d | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 2d<br>1d | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 2d<br>1d | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 1d<br>1d | fabric |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 2w<br>6d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 10d<br>5d | burler, claude |
| E24 |  | **A status-bar button asks for a value, so an agent opened from it gets a name**<br>• Button entries take ask, filling {answer}; the agent button asks for the strand name, and term becomes terminal | [`button-ask-name`](design-button-ask-name.md) | 1d<br>1d | reed, config |

#### Features (15)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>6d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 1d<br>1d | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 1d<br>1d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 1d<br>1d | orch, dobby, arbiter |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 4w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 3d<br>3d | creel, orch |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>3w | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3d<br>3d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 1d<br>1d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>21h | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory and a weight; a worktree picks one when its first session starts, so account A can carry twice the load of B | [`claude-account-weights`](design-claude-account-weights.md) | 3h<br>3h | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>21h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 10d<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>7h | burler, shed, orch, quarry, report |

### Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T3 | **Let tmuxkit start under /dev/shm with a ten-digit uid, then measure fabricengine tier1 there**<br>• Fix checkSocketPath so TMPDIR=/dev/shm works for a ten-digit uid (109 bytes against 107), then run the paired fabricengine tier1 measurement the shm bench could not | [`tmuxkit-short-socket-path`](design-tmuxkit-short-socket-path.md) | 3h<br>2h | testkit, cwd, gate, fabric |
