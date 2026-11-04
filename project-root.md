# P0 Root: entries under no project

Rendered 2026-11-04 15:39 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Hidden-anchor shuttle runs start, and lym shuttle state reads Codex turns**<br>• A hidden-anchor run dies before its provider is ready; every Codex session reads unknown in lym shuttle state | [`shuttle-hidden-and-codex-state`](design-shuttle-hidden-and-codex-state.md) | darn · light | Drift-Verify | neve | 1h<br>17m | shuttle, reed, codex |
| T2 | **Discussion writers start their exploration with explore subagents**<br>• The Discussion-Write stencil asks for one early batch of explore subagents, as the Plan writers already send | [`discussion-explore-agents-first`](design-discussion-explore-agents-first.md) | darn · speed | Finalize | neve | 10m<br>8m | stencil, loom |
| T3 | **Drift accept and skip refuse unknown test names, and the accepted profile is taken at accept**<br>• Two interim Drift repairs: name checks on accept and scope --skip, and the accepted profile recorded when a test is accepted | [`drift-interim-repairs`](design-drift-interim-repairs.md) | darn · light | Code-Bouncer | neve | 33m<br>33m | drift |
| T7 | **Lock files live in .lym, never beside a tracked file in _lym or Bolt**<br>• Locks for _lym and Bolt files sit beside them, hidden only by *.lock excludes; move them under .lym | [`locks-out-of-tracked-trees`](design-locks-out-of-tracked-trees.md) | darn · solid | Darn-Write | neve | 2h<br>1h | fabric, loom |
| T11 | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | darn · light | Darn-Write | neve | 4d<br>1h | pawl, workspaces, reed, landing |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T6 | 3d<br>1h | fabric |

## Notes (26)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 10m<br>10m | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 12d<br>18m | loom, stencil |

### Bugs (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• No waiting line in loom or batten status and no notice while PR-Review waits for a human review | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 20h<br>5h | landing, loom, batten |
| B2 |  | **A weft push holds the remote-refs lock over its whole transfer, so board claims queue behind it**<br>• Bolt and every weft share one remote-refs lock; a slow push stalls every claim and weft fetch for its whole send-pack | [`push-holds-remote-refs`](design-push-holds-remote-refs.md) | 1h<br>1h |  |
| B3 |  | **A test leaves an empty /tmp/.git, which breaks go VCS stamping**<br>• An empty /tmp/.git appeared during a tier1 run and failed TestPrebuildLym; find the leaker and guard against writes outside t.TempDir | [`tmp-git-leak`](design-tmp-git-leak.md) | 1h<br>1h | testkit, gate |

### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 | H | **Drift compares changed values in code against a sealed expectation, with a separate judge**<br>• General Drift rework: code-side diff of any hub capture keyed by line key or number, structured sealed expectation, containment by code, a judge per changed value | [`drift-rework`](design-drift-rework.md) | 14m<br>0m | drift, loom, darn, stencil, config |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 8d<br>3d | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 1d<br>1d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 4d<br>23h | shed, shuttle, batten |
| E13 |  | **The sampler: code writer or read-only discussion role, and the skills that follow** | [`sampler-direction`](design-sampler-direction.md) | 1d<br>1d | loom, stencil |

### Features (16)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>8d | burler |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 3d<br>3d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 3d<br>18m | orch, dobby, arbiter |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 5d<br>5d | creel, orch |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>1d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 5d<br>21h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 3d<br>3d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>3h | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 2d<br>20h | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>21h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 12d<br>18m | loom, board, batten, workspaces |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 1d<br>18m | shuttle, codex, loom |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 1d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>2d | burler, shed, orch, quarry, report |
| F18 |  | **Codex sessions get creel mail through hooks, as Claude sessions do**<br>• codexengine has no creel hooks, so every Codex strand is a pane-mail strand; the hook spike showed a way | [`codex-creel-hooks`](design-codex-creel-hooks.md) | 20h<br>20h | codex, creel, shuttle |
