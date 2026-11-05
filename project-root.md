# P0 Root: entries under no project

Rendered 2026-11-05 19:00 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Slack relay per hub: credentials, setup, inbound and outbound mail, presence and the lym slack verbs** | [`slack-relay`](design-slack-relay.md)<br>[Sketch](decision-slack-relay.md) | tapestry · solid | Webster | neve | 2h<br>1h | reed |
| T7 | **Drift compares changed values in code against a sealed expectation, with a separate judge**<br>• General Drift rework: code-side diff of any hub capture keyed by line key or number, structured sealed expectation, containment by code, a judge per changed value | [`drift-rework`](design-drift-rework.md)<br>[Sketch](decision-drift-rework.md) | tapestry · solid | Parent-Align | neve | 1d<br>5h | drift, loom, darn, stencil, config |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T6 | **Dobby reminds the operator on Slack of a run waiting at PR-Review or Sampler-Session** | [`slack-wait-reminders`](design-slack-wait-reminders.md)<br>[Sketch](decision-slack-wait-reminders.md) | darn · light | T5 | 2h<br>2h | reed |

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **beam merge-parent: the merge-in task can merge, lands as a merge and needs no pull request** | [`merge-parent-lands-right`](design-merge-parent-lands-right.md) | darn · solid | T7 | 1m<br>0m | pawl, darn, landing, fabric |
| T2 | `draft` **Creel mail between hubs: a machine record of hubs, confirmed daemon-to-daemon delivery and replies, per-hub trust and the sender stamp** | [`creel-cross-hub-build`](design-creel-cross-hub-build.md)<br>[Sketch](decision-creel-cross-hub-build.md) | tapestry · solid | T5 | 59m<br>59m | creel, orch |
| T4 | `draft` **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | T7 | 5d<br>5h | fabric |

## Notes (27)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 1d<br>1h | loom, stencil |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1h | loom, stencil |

### Bugs (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• Not a problem now; if built, only after F6 Slack relay, as a Slack reminder ping while a PR-Review waits on the operator | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 1d<br>1h | landing, loom, batten |
| B3 |  | **A finished motif whose Sketch the board refuses stays a draft task with no notice to the orch**<br>• The refusal only logs a Warn; the orch learns of it from a tidy report | [`motif-sketch-refusal-silent`](design-motif-sketch-refusal-silent.md) | 1h<br>1h | loom, board, orch |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 1h<br>1h | reed, orch |
| B5 |  | **A parent cannot withdraw a PATTERN change the decision record declares**<br>• lym loom decision add only appends PATTERN bullets; a declared id the parent drops needs a hand edit of the record | [`decision-withdraw-pattern`](design-decision-withdraw-pattern.md) | 1h<br>1h | loom, parsers, pattern |
| B6 |  | **Raddle on a Sketch fails to start because the motif run that made it already completed under the same slug**<br>• Dobby refuses the raddle run until lym batten cleanup --apply removes the motif run dir | [`raddle-same-slug-completed`](design-raddle-same-slug-completed.md) | 1h<br>1h | batten, dobby, loom |
| B7 |  | **A finished raddle run fails its batten run at Run-Shed** | [`raddle-end-fails-batten`](design-raddle-end-fails-batten.md) | 58m<br>58m | batten |
| B8 |  | **Scope plugins, skills and connectors per role, in every hub**<br>• lym gives each session only the plugins and skills its role needs, ly included in other hubs, and turns off connectors and unused listings | [`ly-plugin-off-in-other-hubs`](design-ly-plugin-off-in-other-hubs.md) | 42m<br>40m | shuttle |
| B10 |  | **webster recover-batch refuses with a way forward that names recover-batch itself** | [`recover-batch-self-way-forward`](design-recover-batch-self-way-forward.md) | 1m<br>1m | webster |
| B11 |  | **webster record-batch: create-not-done refusal lists every searched file and reports a signature mismatch as a missing member** | [`record-batch-signature-mismatch`](design-record-batch-signature-mismatch.md) | 1m<br>1m | webster, quarry |
| B12 |  | **fabric clone --reset removes the hub before refusing a mismatched --shortname** | [`clone-reset-deletes-first`](design-clone-reset-deletes-first.md) | 1m<br>1m | fabric |
| B14 |  | **webster refusal text repeats the full unit file list and every declaration site for each unresolved target** | [`webster-refusal-file-lists`](design-webster-refusal-file-lists.md) | 1m<br>1m | webster, quarry |

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 9d<br>4d | batten, stencil, config |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 2d<br>1h | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 5d<br>2d | shed, shuttle, batten |

### Features (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 4w<br>9d | burler |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>2d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 6d<br>1h | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>1d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 3d<br>1d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1h | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1h | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 2d<br>2d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2w<br>1h | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 13h<br>13h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 13h<br>13h | batten, board, fabric, workspaces |
