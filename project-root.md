# P0 Root: entries under no project

Rendered 2026-11-17 07:19 UTC; every age below is as of then.
Under: [Board](README.md).

Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (5)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam** | [`weaver-implementer`](design-weaver-implementer.md)<br>[Sketch](decision-weaver-implementer.md) | tapestry · solid | Webster | neve | 21h<br>21h | webster, weaver |
| T21 | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | sampler | Sampler-Session | neve | 9d<br>30m | heddle |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>19h | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 4d<br>4d | slack, creel, orch |
| T16 | `draft` **Give Codex lym's shipped heddle instructions too**<br>• Aim lym-batching and lym-plan-edits-before-gate at agents-file and word them for both agents | [`shipped-heddle-agents-file`](design-shipped-heddle-agents-file.md) | darn · light |  | 0m<br>0m | heddle, codex |

## Notes (10)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 7d<br>7d | gate, fabric |
| R4 |  | **Bench Code Mode on a darn writer doing a repeated edit** | [`code-mode-darn-bench`](design-code-mode-darn-bench.md) | 18h<br>18h | claude, shuttle |

### Bugs (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **A capped test run raises a desktop low-memory notice**<br>• Each kernel kill of a capped test scope in app.slice shows a desktop low-memory notice; move the scopes so it does not | [`memcap-desktop-notice`](design-memcap-desktop-notice.md) | 24m<br>24m | gate, process |
| B2 |  | **Memory guard for every test binary lym runs**<br>• The gate watchdog watches CPU only and only gate test runs; give it a memory trigger over every test binary, and cap the gate tiers | [`gate-tier-memory-cap`](design-gate-tier-memory-cap.md) | 1h<br>52m | gate |

### Features (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 1d<br>19h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 6d<br>21h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>11d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>21h | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>11d | burler, shed, orch, quarry, report |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T20 | **gate redundant: a runaway mutant eats all memory and freezes the machine**<br>• Mutant test binaries run four at once with a 10 s timeout floor and no memory cap; a runaway mutant in websterengine stalled the user slice and froze Neve twice<br>• `darn: writer fable` | [`redundant-mutant-memory-cap`](design-redundant-mutant-memory-cap.md) | 1h<br>58m | gate |
