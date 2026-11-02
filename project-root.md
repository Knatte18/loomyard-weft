# P0 Root: entries under no project

Rendered 2026-11-02 19:10 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **lym adopt root leaves the installed lym older than the adopted main** | [`adopt-root-skips-deploy`](design-adopt-root-skips-deploy.md) | darn | Code-Burler-Fix | neve | 34m<br>27m | deploy, fabric, landing |
| T3 | **A wake from hibernate leaves the hub-wide pause live**<br>• After waking, the hibernate request read done but its hub pause stayed live and held every network call; its wake notice failed on a wrong receiver and an oversized line | [`hibernate-wake-leaves-pause`](design-hibernate-wake-leaves-pause.md) | darn | Code-Burler-Fix | neve | 44m<br>27m | hibernate, creel |
| T4 | **Replay the landed plan-split tuning on both bench inputs and tune again** | [`plan-split-replay`](design-plan-split-replay.md) | darn · docs | Worktree-Create | neve | 23m<br>23m | loom, stencil |
| T6 | **Beam orch cannot set a board entry's profile: refused as a task session** | [`beam-orch-cannot-set-profile`](design-beam-orch-cannot-set-profile.md) | darn | Finalize | neve | 7h<br>18m | board, orch, workspaces |
| T7 | **The sampler agent gets no PATTERN directive**<br>• The sampler writes and commits code but its prompt carries no PATTERN directive, unlike every other code-writing role | [`sampler-lacks-pattern-directive`](design-sampler-lacks-pattern-directive.md) | darn | Code-Burler-Fix | neve | 3h<br>13m | loom, stencil, pattern |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (32)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 11d<br>6h | loom, stencil |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B3 |  | **gitexec TestRun_RemoteBounds fails under machine load**<br>• The 200 ms stall bound of the never-answering remote test kills git before its ext script writes its count file when the machine is loaded | [`gitexec-remote-bounds-load-flaky`](design-gitexec-remote-bounds-load-flaky.md) | 5h<br>5h | git, gate |

### Enhancements (16)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Finalize releases the parent lock while its fixer works**<br>• A red verify in Finalize frees the parent lock before the fixer starts, then requeues, merges and verifies again, so green runs land while one run fixes | [`finalize-fix-outside-lock`](design-finalize-fix-outside-lock.md) | 23h<br>23h | landing, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>1d | shuttle, codex |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 6d<br>1d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 4d<br>4d | dotnet, gate |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 1d<br>1d | dobby, orch |
| E6 |  | **A task with no recipe runs darn, not tapestry**<br>• The default recipe becomes darn, set in board.yaml, so a small task promoted without a recipe does not start a full tapestry run | [`default-recipe-darn`](design-default-recipe-darn.md) | 1d<br>1d | board, batten |
| E7 |  | **Operator verbs print a short readable summary, not a JSON dump**<br>• lym adopt root and other verbs the operator types print a long JSON envelope; print a short summary with each stop and its way forward by default, JSON behind a flag | [`operator-verbs-readable-output`](design-operator-verbs-readable-output.md) | 44m<br>44m | cli |
| E8 |  | **The orch role explains how to make a project into a beam**<br>• The prime orch in the Models hub did not know beam create is the only way, and patched a plain task run by hand. | [`orch-role-making-a-beam`](design-orch-role-making-a-beam.md) | 9h<br>9h | orch, pawl |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 2d<br>1d | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 2d<br>2d | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 2d<br>1d | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 2d<br>1d | fabric |
| E13 |  | **The sampler: code writer or read-only discussion role, and the skills that follow** | [`sampler-direction`](design-sampler-direction.md) | 14m<br>14m | loom, stencil |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 2w<br>6d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 11d<br>5d | burler, claude |
| E24 |  | **A status-bar button asks for a value, so an agent opened from it gets a name**<br>• Button entries take ask, filling {answer}; the agent button asks for the strand name, and term becomes terminal | [`button-ask-name`](design-button-ask-name.md) | 1d<br>1d | reed, config |

### Features (14)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>6d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 2d<br>1d | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 1d<br>1d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 2d<br>1d | orch, dobby, arbiter |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 3d<br>3d | creel, orch |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>0m | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3d<br>3d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 1d<br>1d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory and a weight; a worktree picks one when its first session starts, so account A can carry twice the load of B | [`claude-account-weights`](design-claude-account-weights.md) | 6h<br>6h | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 11d<br>1d | loom, board, batten, workspaces |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>10h | burler, shed, orch, quarry, report |
