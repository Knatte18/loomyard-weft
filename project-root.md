# P0 Root: entries under no project

Rendered 2026-11-07 11:31 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **Comment cleanup: parallel agents fix stale comments across the repository**<br>• An orchestrating agent hands each package to a worker, possibly Haiku 5.5, that fixes comments no longer true or pinning perishable specifics | [`comment-cleanup`](design-comment-cleanup.md) | darn · light | Finalize | neve | 6d<br>21h | fabric |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (2)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Comment lint: enforce scribe:prose line-length rule on // comment lines**<br>• lint-comments enforces the line-length rule scribe:prose sets, once it lands | [`comment-lint-line-length`](design-comment-lint-line-length.md) | darn · light | T4 | 2h<br>27m | loom |
| T2 | **Drift fails runs on capture noise such as _source.txt that is no part of the results** | [`drift-provenance-stops-runs`](design-drift-provenance-stops-runs.md) | darn · solid | T4 | 28m<br>22m | drift, loom |

## Notes (30)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Re-measure writer exploration and the carry of lookup results after the Discussion stencil change**<br>• Re-run the writer-lookups measurement on 10 tapestry runs, pricing the carry saved by early explore subagents | [`writer-lookups-remeasure`](design-writer-lookups-remeasure.md) | 2d<br>1d | loom, stencil |
| R3 |  | **Trial creel mail across hubs**<br>• A darn docs run in which the loomyard orch drives a run in lyt from start to landing through creel alone | [`creel-cross-hub-trial`](design-creel-cross-hub-trial.md) | 23h<br>23h |  |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2w<br>1d | loom, stencil |

### Bugs (9)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 | L | `draft` **A PR-Review waiting on the operator shows nowhere**<br>• The Slack reminder ping while a PR-Review waits has landed; only the silence in status views remains, which the operator chose not to build | [`pr-review-wait-silent`](design-pr-review-wait-silent.md) | 3d<br>1d | landing, loom, batten |
| B2 |  | **Batching hook fires on replay but almost never live** | [`batching-hook-live-gap`](design-batching-hook-live-gap.md) | 1d<br>1d | tools, claude |
| B3 |  | **A Codex seat that read its skills is logged as not having loaded them**<br>• All five skills logged not loaded though the rollout shows SKILL.md read; cause not found | [`codex-skill-load-false-missing`](design-codex-skill-load-false-missing.md) | 22h<br>22h | shuttle, codex |
| B4 |  | **A stray bash window appeared in the prime reed session at a prime bring-up**<br>• An unbound bash window opened 37 seconds before the prime overview and orch relaunched; no lym log names it | [`prime-stray-bash-window`](design-prime-stray-bash-window.md) | 1d<br>1d | reed, orch |
| B5 |  | **A session cycle sent /compact with a leading space, so it reached the orch as a plain prompt**<br>• A space in the input box reads as empty, and the slash command then fails silently | [`compact-leading-space`](design-compact-leading-space.md) | 2h<br>2h | shuttle, orch |
| B6 |  | **Subagents spawned by a writer load no language skills, so their comments miss the scribe rules**<br>• Language skills reach the roles lym starts, not the subagents those roles spawn | [`subagents-skip-language-skills`](design-subagents-skip-language-skills.md) | 2h<br>2h | shuttle, stencil |
| B7 |  | **The gate fixer loads only the language skills, not prose or code-quality**<br>• Every code role but the gate fixer starts from scribe:prose, code-quality and testing | [`gate-fixer-no-prose-skills`](design-gate-fixer-no-prose-skills.md) | 2h<br>2h | gate, loom |
| B8 |  | **lym shuttle open sessions write a 1h prompt cache instead of their role's 5m**<br>• The open path seems not to set CLAUDE_CODE_PROMPT_CACHE_TTL | [`shuttle-open-cache-ttl`](design-shuttle-open-cache-ttl.md) | 1h<br>1h | shuttle, claude |
| B9 |  | **Subagents spawned without an effort run at xhigh, not the medium settings name**<br>• Haiku workers with no effort in the Agent call ran at xhigh | [`subagent-default-effort-xhigh`](design-subagent-default-effort-xhigh.md) | 1h<br>1h | shuttle, claude, stencil |

### Enhancements (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **The gate builds and starts test binaries for packages with no test of the tier**<br>• Pass only the packages holding a test of the tier to go test; weighs most under Cortex XDR | [`gate-tier-binaries-only`](design-gate-tier-binaries-only.md) | 2h<br>2h | gate |
| E2 |  | **git worktree list is 15 percent of tier 1 git processes and only reads .git/worktrees**<br>• Serve the list from dotgit, with a parity test against git | [`worktree-list-through-dotgit`](design-worktree-list-through-dotgit.md) | 2h<br>2h | fabric |
| E3 |  | **A run keeps the lym binary and Bolt instructions it started with** | [`run-pins-its-version`](design-run-pins-its-version.md) | 10d<br>1d | batten, stencil, config |
| E4 |  | **trusted_hubs lists shortnames, which neither say which repository is meant nor identify it**<br>• List trusted hubs by full repository name instead of shortname | [`creel-trust-by-repo-name`](design-creel-trust-by-repo-name.md) | 2h<br>2h | creel |
| E7 |  | **Each part agent plans and builds its own part, instead of joining the parts into one plan** | [`parts-plan-and-build`](design-parts-plan-and-build.md) | 4d<br>1d | loom |
| E9 |  | **An interactive agent idle for a set time gets its session compacted**<br>• lym asks an idle interactive agent to commit what it has and end, and the run resumes later | [`interactive-idle-park`](design-interactive-idle-park.md) | 6d<br>3d | shed, shuttle, batten |

### Features (12)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 5w<br>11d | burler |
| F2 |  | **Slack presence follows where the operator writes, with a reed toggle**<br>• lym switches presence on an operator Slack message or pane line, tells operator lines from its own injects, and shows a slack on/off toggle in reed | [`slack-presence-auto`](design-slack-presence-auto.md) | 1d<br>1d | reed, shuttle |
| F8 |  | **fabric reset: reset warp to a SHA and re-anchor the weft trailers**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 4w<br>4d | fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 8d<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 4w<br>2d | board, ide, orch |
| F12 |  | **Two Claude accounts, each worktree on one of them by a configured weight**<br>• A claude config lists accounts with a config directory, a weight and their limit windows; a worktree picks one at its first session by weight and current headroom | [`claude-account-weights`](design-claude-account-weights.md) | 4d<br>3d | claude, shuttle, config |
| F13 | L | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 3w<br>1d | webster |
| F14 |  | **The decision record on the board, and splitting a discussed task**<br>• A task flagged discussed carries its decision record on the board and skips Discussion; three ways to split, from by hand to stitch | [`discussed-task-split`](design-discussed-task-split.md) | 2w<br>1d | loom, board, batten, workspaces |
| F16 |  | **Drift cannot be trialed in a sampler: every lym drift verb refuses outside drift stages** | [`sampler-drift-trial`](design-sampler-drift-trial.md) | 4d<br>1d | drift, shed |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3w<br>1d | burler, shed, orch, quarry, report |
| F19 |  | **A beam orch cannot create a project at the root, only under its own beam** | [`beam-orch-root-project`](design-beam-orch-root-project.md) | 2d<br>23h | board, orch, workspaces |
| F20 |  | **Move a running task to another parent (beam or prime)** | [`reparent-running-task`](design-reparent-running-task.md) | 2d<br>2d | batten, board, fabric, workspaces |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T3 | **Revise tier 1: what makes it slow, and which tests to change, fold or cut**<br>• Ranked report on tier 1, fabricengine first; tier 2 stays in the gate keys | [`tier1-test-revision`](design-tier1-test-revision.md) | 22h<br>22h | fabric |
