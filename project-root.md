# P0 Root: entries under no project

Rendered 2026-11-03 09:43 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (7)

Concrete and claimable; only a task can run.

### Running (5)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Tapestry runs Discussion through a motif block and skips it for a discussed entry; motif runs alone; arras goes**<br>• Tapestry uses the motif block, skips it when the entry is flagged discussed, motif becomes a runnable recipe, and arras is removed | [`tapestry-motif-block`](design-tapestry-motif-block.md) | darn · solid | Darn-Write | neve | 1m<br>1m | shed, loom |
| T3 | **Batten recovers a run whose pair is gone and orders its remote pushes, so no pass fails forever and no push races another** | [`batten-remote-order`](design-batten-remote-order.md) | tapestry · light | Plan-Burler-Review | neve | 13h<br>2h | batten, fabric |
| T4 | **Drift scope settable in a task's run_settings; Drift-Baseline agent only when unset** | [`drift-scope-in-run-settings`](design-drift-scope-in-run-settings.md) | darn · solid | Code-Burler-Review | neve | 14m<br>14m | drift, config |
| T5 | **Preflight refuses lym own state files under _lym**<br>• Commit or exempt every file lym writes under _lym on a stop path, such as go-settle.jsonl | [`preflight-clean-skips-lym-state`](design-preflight-clean-skips-lym-state.md) | darn · solid | Code-Bouncer | neve | 42m<br>42m | preflight, shed |
| T6 | **config reconcile never migrates profiles.yaml, so a hub seeded before the loom-to-tapestry rename stops tapestry runs at Seed-Child** | [`reconcile-migrates-profiles`](design-reconcile-migrates-profiles.md) | darn · solid | Code-Burler-Review | neve | 27m<br>27m | reconcile, profiles, config |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | **fabric remove deletes the whole worktree directory, ignored files included** | [`fabric-remove-ignored-files`](design-fabric-remove-ignored-files.md) | darn · solid | T3 | 1h<br>1h | fabric |
| T8 | **Batten treats a stray non-worktree directory as a gone pair, and a beam landing accepts its own beam mark** | [`batten-stray-dir-and-beam-mark`](design-batten-stray-dir-and-beam-mark.md) | darn · solid | T3 | 1h<br>1h | batten, pawl |

## Notes (41)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 11d<br>2h | loom, stencil |

### Bugs (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B2 |  | **A paused run whose work already landed is never finished and keeps its worktree and terminal** | [`landed-run-left-paused`](design-landed-run-left-paused.md) | 13h<br>13h | dobby, batten |
| B3 |  | **Darn-Write seat halts: stencil darn-template-write has unfilled marker subagent_model** | [`seat-stencil-marker-unfilled`](design-seat-stencil-marker-unfilled.md) | 13h<br>2h | stencil, shed |
| B5 |  | **validate-plan edit-test-file rule cannot target a test file with no glyphs** | [`edit-test-file-no-glyphs`](design-edit-test-file-no-glyphs.md) | 13h<br>13h | loom, quarry |
| B6 |  | **insert-card renumbers only <N>-<slug> ids, leaving bare 'card N' mentions stale** | [`insert-card-bare-mentions`](design-insert-card-bare-mentions.md) | 13h<br>13h | loom |
| B8 |  | **Run ends with a background shell outstanding: its commit and gate result are lost** | [`run-done-with-background-shell`](design-run-done-with-background-shell.md) | 13h<br>13h | shuttle, webster, gate |
| B12 |  | **lym config drift refuses the whole-map capture and compare form its own key description prescribes** | [`drift-config-whole-map-set`](design-drift-config-whole-map-set.md) | 2h<br>2h | drift, config |

### Enhancements (16)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **A wake from hibernate checks what moved on origin and lets runs go on only when nothing did and a switch allows it** | [`hibernate-wake-resume-switch`](design-hibernate-wake-resume-switch.md) | 14h<br>13h | hibernate, tender, fabric |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 6d<br>1d | batten, stencil, config |
| E4 |  | **Prove the dotnet scoper, shared build and Drift on the C# sandbox hub**<br>• Switch the sandbox gate to scoper dotnet, a build step and --no-build, then enable Drift there | [`sandbox-dotnet-gate`](design-sandbox-dotnet-gate.md) | 4d<br>52m | dotnet, gate |
| E5 |  | **A landing notice is not lost when set late, when dobby was down, or when the orch is absent**<br>• Close the known limits landing-watch left: a late notify_on_land, a Land passed while dobby was down, and a dropped line when no orch is live | [`landing-watch-limits`](design-landing-watch-limits.md) | 2d<br>2d | dobby, orch |
| E6 |  | **Plan rubric's test-economy rule conflicts with decision-record acceptance criteria phrased as rendered-output checks** | [`rubric-test-economy-vs-criteria`](design-rubric-test-economy-vs-criteria.md) | 13h<br>13h | burler, loom, stencil |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 12h<br>12h | loom |
| E8 |  | **The orch role explains how to make a project into a beam**<br>• The prime orch in the Models hub did not know beam create is the only way, and patched a plain task run by hand. | [`orch-role-making-a-beam`](design-orch-role-making-a-beam.md) | 23h<br>23h | orch, pawl |
| E9 |  | **An idle interactive run saves its work and pauses, to go on another time**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 2d<br>2d | shed, shuttle, batten |
| E10 |  | **The hibernate request is carried by dobby, not a daemon of its own**<br>• Move lym hibernate request into dobby; the past-Land test is already shared | [`hibernate-in-dobby`](design-hibernate-in-dobby.md) | 2d<br>2d | dobby |
| E11 |  | **Merge in from parent as a task of its own in a beam, before pawl lands it**<br>• A command on a beam files a merge-in-from-parent task, run as darn with its perch on | [`parent-merge-task`](design-parent-merge-task.md) | 2d<br>1d | pawl, workspaces, drift, landing |
| E12 |  | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | 2d<br>1d | fabric |
| E13 |  | **The sampler: code writer or read-only discussion role, and the skills that follow** | [`sampler-direction`](design-sampler-direction.md) | 14h<br>13h | loom, stencil |
| E14 |  | **A beam session shows the pawl run checklist in a second overview window beside the beam overview** | [`pawl-checklist-hidden-in-beam`](design-pawl-checklist-hidden-in-beam.md) | 2h<br>2h | pawl, orch, reed |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 2w<br>6d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 11d<br>5d | burler, claude |
| E24 |  | **A status-bar button asks for a value, so an agent opened from it gets a name**<br>• Button entries take ask, filling {answer}; the agent button asks for the strand name, and term becomes terminal | [`button-ask-name`](design-button-ask-name.md) | 1d<br>1d | reed, config |

### Features (18)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>6d | burler |
| F2 |  | **A status-bar button opens a short-lived board agent, so board talks need no orch**<br>• An ordinary agent with a board-policy skill loaded, for discussing and filing notes and releasing tasks on go-ahead | [`board-agent-button`](design-board-agent-button.md) | 2d<br>1d | reed, board, shuttle, orch |
| F3 | L | **A module overlap lets a task run Discussion at once and wait only before Plan**<br>• A second kind of dependency, plan_after: the task starts and discusses, then waits before Plan until the named tasks have landed | [`plan-after-dependency`](design-plan-after-dependency.md) | 1d<br>1d | board, batten, loom |
| F4 |  | **The orch lies dormant and wakes only when a run needs help**<br>• Dobby takes the mechanics and a spawned agent the doubtful cases; one orch per beam lives on, dormant, woken when needed | [`dormant-orch`](design-dormant-orch.md) | 2d<br>2d | orch, dobby, arbiter |
| F5 |  | **Per-machine environment variables, exported to every process lym starts for the hub** | [`machine-env-vars`](design-machine-env-vars.md) | 2h<br>2h | config, gate, drift |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 4w<br> | reed |
| F7 |  | **Creel mail between hubs through an inbox folder, with a verified sender**<br>• A shared inbox folder takes mail from other hubs; each file names the sender hub and agent so a reply knows where to go | [`creel-cross-hub-mail`](design-creel-cross-hub-mail.md) | 4d<br>4d | creel, orch |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>14h | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 4d<br>4d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F10 |  | **The arbiter settles review escalations and records their decisions, so they never reach the orch**<br>• Decision escalations become eligible for the arbiter, and the arbiter may record decisions and accept or continue a review | [`arbiter-settles-escalations`](design-arbiter-settles-escalations.md) | 1d<br>1d | burler, arbiter, dobby, orch |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 21h<br>17m | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 11d<br>1h | loom, board, batten, workspaces |
| F15 |  | **Writer lookup agents on another engine, such as Codex, spawned through shuttle as reed windows** | [`lookup-agents-through-shuttle`](design-lookup-agents-through-shuttle.md) | 2h<br>2h | shuttle, codex, loom |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 27m<br>27m | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>1d | burler, shed, orch, quarry, report |
| F18 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 4w<br>9m | shuttle, codex |
