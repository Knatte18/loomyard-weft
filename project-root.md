# P0 Root: entries under no project

Rendered 2026-11-04 11:18 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (9)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T7 | **Beam pull requests: reopen a beam from PR-Review, name PRs in squash commits, list sub-PRs**<br>• lym beam reopen back to Beam-Await, PR number in squash subjects, Beam-Describe lists the sub-task PRs | [`beam-pr-lifecycle`](design-beam-pr-lifecycle.md) | darn · solid | Record-Write | neve | 1h<br>56m | pawl, landing, cli |
| T8 | **A shuttle session gets its prompt and its creel hooks at launch**<br>• Darn-Write never got its shuttle prompt; a lym shuttle open session gets no creel hooks | [`shuttle-launch-delivery`](design-shuttle-launch-delivery.md) | darn · light | Worktree-Create | neve | 15h<br>56m | shuttle, claude, creel, darn, loom |
| T9 | **Drift capture fails by name with no baseline, and a reinterpreted drift.yaml is migrated**<br>• A capture that writes no baseline fails with a cause; a binary that moves the dir base migrates drift.yaml | [`drift-capture-and-migration`](design-drift-capture-and-migration.md) | darn · light | Worktree-Create | neve | 18h<br>56m | drift, loom, reconcile |
| T12 | **Dobby and the orch watcher leave their child processes as zombies**<br>• Hundreds of defunct lym children pile up under dobby and orch watch, and tender lists them as live | [`dobby-zombie-children`](design-dobby-zombie-children.md) | darn · light | Worktree-Create | neve | 15h<br>56m | dobby, orch, tender, process |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T13 | **loom drive writes untracked files under _lym/; _lym is expected to be fully tracked** | [`drive-reports-untracked`](design-drive-reports-untracked.md) | darn · light |  | 1h<br>56m | loom, shed, fabric |

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T10 | **Let the pr-reviewer read and edit on GitHub** | [`pr-reviewer-github-access`](design-pr-reviewer-github-access.md) | darn · light | T7 | 1h<br>56m | landing, stencil |
| T11 | **A crashed test runner is treated as a failing gate and spends a fixer**<br>• Exit 134 with no test result sends Finalize to an agent fixer on a green tree instead of a rerun | [`gate-runner-crash-fixer`](design-gate-runner-crash-fixer.md) | darn · light | T7 | 15h<br>56m | gate, landing |

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Review Drift as a whole and redesign it from the trials**<br>• Two C# trials found many Drift faults, mostly in agent judgment and gaps in the flow; map the whole and propose one design | [`drift-whole-review`](design-drift-whole-review.md) | darn · docs |  | 16h<br>15h | drift, stencil, loom |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T1<br>T3<br>T5<br>T6<br>T7<br>T8<br>T9<br>T10<br>T11<br>T12<br>T13 | 3d<br>56m | fabric |

## Notes (28)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 12d<br>1d | loom, stencil |

### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• No waiting line in loom or batten status and no notice while PR-Review waits for a human review | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 16h<br>56m | landing, loom, batten |
| B2 |  | **Worktree-Create runs one at a time and spends minutes on network round trips**<br>• Five released tasks left Worktree-Create one by one over minutes, behind fetches, remote-ref reads and a weft push | [`worktree-create-serial`](design-worktree-create-serial.md) | 0m<br>0m | fabric, batten, dobby |

### Enhancements (7)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **lym ide opens configured files on start, rendered markdown for a beam README**<br>• A beam IDE starts with its README rendered; which files open is config, not code, so per-beam docs can be named later | [`ide-open-files`](design-ide-open-files.md) | 16h<br>16h | ide, config |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 7d<br>2d | batten, stencil, config |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 3d<br>3d | dobby, orch |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 1d<br>1d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 3d<br>18h | shed, shuttle, batten |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 3d<br>17h | pawl, workspaces, reed, landing |
| E13 |  | **The sampler: code writer or read-only discussion role, and the skills that follow** | [`sampler-direction`](design-sampler-direction.md) | 1d<br>1d | loom, stencil |

### Features (18)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>8d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 3d<br>16h | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 2d<br>2d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 3d<br>3d | orch, dobby, arbiter |
| F5 |  | **A run lands on a parent the hub may not push to by merging on the remote**<br>• Branch protection refuses a direct push to main, so no task or beam can land on such a repository | [`protected-parent-landing`](design-protected-parent-landing.md) | 17h<br>17h | landing, git |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 5d<br>5d | creel, orch |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>1d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 5d<br>16h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 2d<br>2d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>23h | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 1d<br>16h | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>16h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 12d<br>19h | loom, board, batten, workspaces |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 1d<br>18h | shuttle, codex, loom |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 1d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>2d | burler, shed, orch, quarry, report |
| F18 |  | **Codex sessions get creel mail through hooks, as Claude sessions do**<br>• codexengine has no creel hooks, so every Codex strand is a pane-mail strand; the hook spike showed a way | [`codex-creel-hooks`](design-codex-creel-hooks.md) | 15h<br>15h | codex, creel, shuttle |
