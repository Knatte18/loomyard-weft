# P0 Root: entries under no project

Rendered 2026-11-06 13:13 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **The python guard refuses without saying why or what to do when a task needs Python**<br>• An OCR repository on lym hits the guard: the refusal gives no reason and no way forward | [`python-guard-gives-no-reason`](design-python-guard-gives-no-reason.md) | darn · light | Darn-Write | neve | 42m<br>37m | shuttle |
| T2 | **A skill load stopped by an API error counts as skipped, and the prompt is sent into the outage**<br>• Shuttle skips unloaded skills with only a log line; an API outage leaves a seat stalled or running without them | [`skill-load-timeout-sends-prompt`](design-skill-load-timeout-sends-prompt.md) | darn · light | Darn-Write | neve | 15m<br>12m | shuttle |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light |  | 5d<br>23h | fabric |

## Notes (22)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 1d<br>20h | loom, stencil |
| R3 |  | **Trial creel mail across hubs**<br>• A darn docs run in which the loomyard orch drives a run in lyt from start to landing through creel alone | [`creel-cross-hub-trial`](design-creel-cross-hub-trial.md) | 1h<br>1h |  |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>17h | loom, stencil |

### Bugs (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• The Slack reminder ping while a PR-Review waits has landed; only the silence in status views remains, which the operator chose not to build | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 2d<br>3h | landing, loom, batten |
| B2 |  | **Batching hook fires on replay but almost never live** | [`batching-hook-live-gap`](design-batching-hook-live-gap.md) | 17h<br>17h | tools, claude |
| B3 |  | **A Codex seat that read its skills is logged as not having loaded them**<br>• All five skills logged not loaded though the rollout shows SKILL.md read; cause not found | [`codex-skill-load-false-missing`](design-codex-skill-load-false-missing.md) | 8m<br>8m | shuttle, codex |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 20h<br>20h | reed, orch |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 9d<br>17h | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 3d<br>20h | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 5d<br>2d | shed, shuttle, batten |

### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>10d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 16h<br>16h | reed, shuttle |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>3d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 7d<br>20h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>2d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 4d<br>2d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>20h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>20h | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 3d<br>17h | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>20h | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 1d<br>1h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 1d<br>1d | batten, board, fabric, workspaces |

## Done (2)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T5 | **A fanned assistant seat is told to work alone, and ran without its lenses**<br>• The composed prompt forbids what the fan asks for; the real seat went solo and no record flagged it<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`assist-fan-seat-work-alone`](design-assist-fan-seat-work-alone.md) | 1h<br>56m | burler, stencil, codex |
| T7 | **Shorten the orch role file, the largest single start cost**<br>• role.md costs 19,390 tokens per read: shorten it, or move rarely used parts to a file the orch reads on demand<br>• `drift: scope off` | [`orch-role-shorter`](design-orch-role-shorter.md) | 16h<br>3h | orch |
