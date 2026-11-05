# P0 Root: entries under no project

Rendered 2026-11-05 06:27 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (7)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Motif runs: own profile recipe, a finished state on the board, and live status**<br>• Motif drops profile_recipe tapestry, a finished motif reads as done, and the board status follows batten rows and the inner producer | [`motif-board-fixes`](design-motif-board-fixes.md) | darn · light | Darn-Write | neve | 36m<br>34m | loom, board, batten, profiles |
| T3 | **Loop exited without an envelope while step Webster was in flight** | [`loop-exit-no-envelope-cause`](design-loop-exit-no-envelope-cause.md) | darn · light | Finalize | neve | 36m<br>34m | shed, friction |
| T7 | **Lock files live in .lym, never beside a tracked file in _lym or Bolt**<br>• Locks for _lym and Bolt files sit beside them, hidden only by *.lock excludes; move them under .lym | [`locks-out-of-tracked-trees`](design-locks-out-of-tracked-trees.md) | darn · solid | Code-Bouncer | neve | 16h<br>15h | fabric, loom |
| T9 | **A test leaves an empty /tmp/.git, which breaks go VCS stamping**<br>• An empty /tmp/.git appeared during a tier1 run and failed TestPrebuildLym; find the leaker and guard against writes outside t.TempDir | [`tmp-git-leak`](design-tmp-git-leak.md) | darn · light | Darn-Write | neve | 15h<br>23m | testkit, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Teardown removes the whole worktree directory, and a gone pair blocks no board edit**<br>• Worktree-Teardown and fabric remove delete leftover build output; has-a-pair and batten cancel treat a non-git directory as no pair | [`teardown-gone-pair`](design-teardown-gone-pair.md) | darn · light | T7<br>T1 | 36m<br>34m | batten, fabric, board |
| T8 | **The beam orch drives the merge-in, pull request and review of its beam inside Beam-Await**<br>• Pawl drops Parent-Align, Beam-Describe and PR-Review; the beam orch opens and closes the PR, merges the parent in and runs lym beam ready | [`pawl-orch-landing`](design-pawl-orch-landing.md) | tapestry · solid | T7 | 42m<br>26m | pawl, landing, orch, workspaces, fabric |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T6 | 4d<br>16h | fabric |

## Notes (30)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 14h<br>39m | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 13d<br>15h | loom, stencil |

### Bugs (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• No waiting line in loom or batten status and no notice while PR-Review waits for a human review | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 1d<br>20h | landing, loom, batten |
| B2 |  | **A weft push holds the remote-refs lock over its whole transfer, so board claims queue behind it**<br>• Bolt and every weft share one remote-refs lock; a slow push stalls every claim and weft fetch for its whole send-pack | [`push-holds-remote-refs`](design-push-holds-remote-refs.md) | 16h<br>16h |  |
| B3 |  | **An opened Codex session never reads idle, so creel types nothing into it**<br>• SessionIdle judges a strand with no run record by the hub default engine, so a Codex pane reads as no input box | [`opened-codex-never-idle`](design-opened-codex-never-idle.md) | 4m<br>4m | shuttle, creel, codex |
| B4 |  | **lym shuttle state keeps a Codex run asking after Codex exits**<br>• After /exit the pane is dead and the loop ends died, while the session state still reads asking | [`codex-state-asking-after-exit`](design-codex-state-asking-after-exit.md) | 4m<br>4m | shuttle, codex |
| B5 |  | **A Codex session cannot reach the creel daemon from its workspace-write sandbox**<br>• lym creel read and post fail inside Codex: connect to the daemon socket is not permitted and the daemon log is read-only | [`codex-sandbox-no-creel-socket`](design-codex-sandbox-no-creel-socket.md) | 4m<br>4m | codex, creel, shuttle |

### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 | H | **Drift compares changed values in code against a sealed expectation, with a separate judge**<br>• General Drift rework: code-side diff of any hub capture keyed by line key or number, structured sealed expectation, containment by code, a judge per changed value | [`drift-rework`](design-drift-rework.md) | 15h<br>39m | drift, loom, darn, stencil, config |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 8d<br>3d | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 2d<br>2d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 4d<br>1d | shed, shuttle, batten |
| E13 |  | **The sampler: code writer or read-only discussion role, and the skills that follow** | [`sampler-direction`](design-sampler-direction.md) | 2d<br>2d | loom, stencil |

### Features (18)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>8d | burler |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 3d<br>3d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 4d<br>15h | orch, dobby, arbiter |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 5d<br>5d | creel, orch |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>2d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 5d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 3d<br>3d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>18h | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 2d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 13d<br>15h | loom, board, batten, workspaces |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 1d<br>39m | shuttle, codex, loom |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 1d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>2d | burler, shed, orch, quarry, report |
| F18 |  | **Codex sessions get creel mail through hooks, as Claude sessions do**<br>• codexengine has no creel hooks, so every Codex strand is a pane-mail strand; the hook spike showed a way | [`codex-creel-hooks`](design-codex-creel-hooks.md) | 1d<br>1d | codex, creel, shuttle |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 36m<br>36m | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 36m<br>36m | batten, board, fabric, workspaces |
