# P0 Root: entries under no project

Rendered 2026-11-07 12:50 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (9)

Concrete and claimable; only a task can run.

### Running (7)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Drift fails runs on capture noise such as _source.txt that is no part of the results** | [`drift-provenance-stops-runs`](design-drift-provenance-stops-runs.md) | darn · solid | Finalize | neve | 1h<br>1h | drift, loom |
| T3 | **Claude sessions: a slash command sent with a leading space, and open sessions caching 1h**<br>• Two claudeengine bugs: /compact as a prompt, and shuttle open without the 5m TTL | [`claude-session-fixes`](design-claude-session-fixes.md) | darn · light | Finalize | neve | 3h<br>46m | shuttle, claude |
| T4 | **fabric clone leaves the prime without heddle's rendered files**<br>• Made fires before the config reconcile and its warning is dropped | [`clone-prime-heddle-unrendered`](design-clone-prime-heddle-unrendered.md) | darn · light | Drift-Recheck | neve | 53m<br>46m | fabric, heddle |
| T5 | **The skill-load hold landed with two checks by hand left open** | [`skill-load-hold-unchecked`](design-skill-load-hold-unchecked.md) | darn · light | Drift-Await | neve | 52m<br>0m | shuttle |
| T6 | **A Codex seat that read its skills is logged as not having loaded them**<br>• All five skills logged not loaded though the rollout shows SKILL.md read; cause not found | [`codex-skill-load-false-missing`](design-codex-skill-load-false-missing.md) | darn · light | Finalize | neve | 23h<br>46m | shuttle, codex |
| T7 | **Batching hook fires on replay but almost never live** | [`batching-hook-live-gap`](design-batching-hook-live-gap.md) | darn · light | Finalize | neve | 1d<br>46m | tools, claude |
| T9 | **The gate builds and starts test binaries for packages with no test of the tier**<br>• Pass only the packages holding a test of the tier to go test; weighs most under Cortex XDR | [`gate-tier-binaries-only`](design-gate-tier-binaries-only.md) | darn · light | Darn-Write | neve | 4h<br>1m | gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Heddle renders .claude/settings.local.json in every hub: no co-author, lym's plugins and tool deny list**<br>• Attribution off and lym model/effort defaults, in session settings and by an operator-run setup verb | [`machine-claude-defaults`](design-machine-claude-defaults.md) | darn · solid | T4 | 39m<br>4m | claude, config, heddle, fabric |

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Skills and effort for every role and subagent come from config**<br>• Skills per role in config, gate fixer included; subagents load fitting skills; every effort explicit | [`agent-skills-and-effort`](design-agent-skills-and-effort.md) | darn · solid | T3<br>T1 | 3h<br>4m | shuttle, stencil, claude, gate, loom |

## Notes (23)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 2d<br>1d | loom, stencil |
| R3 |  | **Trial creel mail across hubs**<br>• A darn docs run in which the loomyard orch drives a run in lyt from start to landing through creel alone | [`creel-cross-hub-trial`](design-creel-cross-hub-trial.md) | 1d<br>1d |  |
| R5 |  | **The Python guard's new refusal text was never seen in a real session** | [`python-guard-live-unchecked`](design-python-guard-live-unchecked.md) | 52m<br>52m | shuttle, claude |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1d | loom, stencil |
| R7 |  | **Re-measure the batching hook once its transcript wait has landed**<br>• Count firings against edit follows and lookup chains again, now that the hook waits for its call to reach the transcript | [`batching-hook-remeasure-wait`](design-batching-hook-remeasure-wait.md) | 39m<br>39m | claude, shuttle |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 1d<br>1d | reed, orch |

### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **git worktree list is 15 percent of tier 1 git processes and only reads .git/worktrees**<br>• Serve the list from dotgit, with a parity test against git | [`worktree-list-through-dotgit`](design-worktree-list-through-dotgit.md) | 4h<br>4h | fabric |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 10d<br>1d | batten, stencil, config |
| E4 |  | **trusted_hubs lists shortnames, which neither say which repository is meant nor identify it**<br>• List trusted hubs by full repository name instead of shortname | [`creel-trust-by-repo-name`](design-creel-trust-by-repo-name.md) | 4h<br>4h | creel |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 4d<br>1d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 6d<br>3d | shed, shuttle, batten |

### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>11d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 1d<br>1d | reed, shuttle |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>4d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 8d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>3d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 5d<br>3d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 4d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>1d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 2d<br>1h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 2d<br>2d | batten, board, fabric, workspaces |
