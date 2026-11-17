# P0 Root: entries under no project

Rendered 2026-11-17 06:49 UTC; every age below is as of then.
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
| T5 | **Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam** | [`weaver-implementer`](design-weaver-implementer.md)<br>[Sketch](decision-weaver-implementer.md) | tapestry · solid | Webster | neve | 20h<br>20h | webster, weaver |
| T16 | **Retro of 10 landed runs up to 2026-11-16T11:54:11Z**<br>• Audit the runs from their records and transcripts, and propose fixes, most severe first | [`retro-20261116-115411`](design-retro-20261116-115411.md) | darn · docs | Darn-Write | neve | 2m<br>2m |  |
| T20 | **gate redundant: a runaway mutant eats all memory and freezes the machine**<br>• Mutant test binaries run four at once with a 10 s timeout floor and no memory cap; a runaway mutant in websterengine stalled the user slice and froze Neve twice<br>• `darn: writer fable` | [`redundant-mutant-memory-cap`](design-redundant-mutant-memory-cap.md) | darn · light | Darn-Write | neve | 41m<br>28m | gate |
| T21 | **A sampler that iterates the rendering of HEDDLE.md until it reads well** | [`heddle-md-readability`](design-heddle-md-readability.md) | sampler | Record-Write | neve | 9d<br>0m | heddle |
| T22 | **lym retro: run the retro task's writer on Opus**<br>• The retro task's writer judges transcripts and routes proposals; give it Opus instead of the docs default | [`retro-writer-opus`](design-retro-writer-opus.md) | darn · docs | Preflight | neve | 1m<br>0m | report, darn |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Two Claude accounts, chosen per worktree, with orchs and beams pinnable to one account**<br>• Revised design of claude-account-weights without a per-worktree record; reuses its decision record and body | [`claude-account-pick-simple`](design-claude-account-pick-simple.md) | motif | P4 | 2w<br>18h | claude, shuttle, config |
| T7 | `draft` **Build the Slack bridge that carries creel mail between orchs on two machines** | [`slack-machine-bridge-build`](design-slack-machine-bridge-build.md)<br>[Sketch](decision-slack-machine-bridge-build.md) | tapestry · solid |  | 4d<br>4d | slack, creel, orch |

## Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 7d<br>7d | gate, fabric |
| R4 |  | **Bench Code Mode on a darn writer doing a repeated edit** | [`code-mode-darn-bench`](design-code-mode-darn-bench.md) | 17h<br>17h | claude, shuttle |

### Bugs (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B2 |  | **Memory guard for every test binary lym runs**<br>• The gate watchdog watches CPU only and only gate test runs; give it a memory trigger over every test binary, and cap the gate tiers | [`gate-tier-memory-cap`](design-gate-tier-memory-cap.md) | 30m<br>23m | gate |

### Features (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 6w<br>2w | burler |
| F3 |  | **Move a running task by hand-back and a fresh run, not a re-parent verb**<br>• The writer hands back what it knows, the run is cancelled, a new task under the new parent starts from that | [`move-task-by-handback`](design-move-task-by-handback.md) | 1d<br>18h | batten, loom, board |
| F4 |  | **config: per-machine or per-user branch_prefix, and a review of other per-user keys** | [`per-machine-branch-prefix`](design-per-machine-branch-prefix.md) | 6d<br>21h | config, fabric |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 2w<br>11d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 6w<br>21h | board, ide, orch |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>11d | burler, shed, orch, quarry, report |
