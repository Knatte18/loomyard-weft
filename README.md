# Board

Rendered 2026-10-25 12:39 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Projects (2)

Groups of entries; a beam spawns its members from its own worktree.

### P1 Workspaces: beams, pairs from a parent pair, dobby

Beams that develop a project in their own worktree, driven by a dobby daemon

grouping

| # | Member | Kind | Standing |
| --- | --- | --- | --- |
| T3 | **An anchor per beam** | Task | Loom-Preflight |

### P2 .NET: LoomYard for C# repositories

Running LoomYard on C#/.NET repositories such as Models: scoped gates, regression, a sandbox hub

grouping

| # | Member | Kind | Standing |
| --- | --- | --- | --- |
| F18 | **Regression as three Drift rows: Drift-Baseline after Discussion, Drift-Await before Webster, Drift-Verify after the Code perch** | Note | Note |
| E28 | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | Note | Note |

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **An anchor per beam**<br>• Let each worktree carry its own anchor through a .lyx-anchor marker at its weft branch root, inherited by its tasks, with a verb that changes it<br>• `profile: solid` | [`umbrella-anchor`](design-umbrella-anchor.md) | loom | Loom-Preflight | neve | 6d<br>5m | fabric, config, workspaces |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | `draft` **Bench Codex Astra against solo Opus as a plan reviewer, four runs**<br>• `profile: docs` | [`codex-astra-solo-bench`](design-codex-astra-solo-bench.md) | darn |  | 23m<br>23m | codex, burler |

## Notes (35)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits (6)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| A2 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 6d<br>21h |  |
| A3 |  | **Audit the twill and lens review setup: worth per perch, recurring finding classes, writer quality**<br>• One audit over the review records once about ten loom runs with the current setup land on Neve | [`review-setup-audit`](design-review-setup-audit.md) | 9d<br>21h | burler, loom, report, stencil, shed |
| A5 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 5d<br>5d |  |
| A6 |  | **Lens overlap audit: which lenses find the same defects, and a smaller set**<br>• twill-plan 8 to 5 lenses recommended; twill-discussion 7 to 4 and review-code 4 to 3 at low confidence | [`lens-overlap-audit`](design-lens-overlap-audit.md) | 2d<br>39m | burler, loom |
| A7 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 2d<br>1d | webster |
| A8 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 2d<br>1d | loom, stencil |

### Bugs (1)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Find why a running run shows a dead waiter in the activity reader** | [`idle-notice-dead-waiter-cause`](design-idle-notice-dead-waiter-cause.md) | 1d<br>1d | batten, shuttle |

### Enhancements (14)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>35m | shuttle, codex |
| E4 |  | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w<br>2w | fabric, loom |
| E5 |  | **Decide the layout of projects in the board README**<br>• The operator is not satisfied with how the README renders projects and their members; the layout is to be decided | [`board-projects-layout`](design-board-projects-layout.md) | 6h<br>6h | board |
| E7 |  | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 10d<br>39m | shed, loom |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 13d<br>1d | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 10d<br>10d | board, loom |
| E13 |  | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 9d<br>3d | stencil, shuttle |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 7d<br>7d | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 6d<br>6d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 2d<br>2d | burler, claude |
| E27 | L | **Creel delivers agent mail in full at the receiver's next tool boundary**<br>• A reminder plus lyx creel read costs the receiver an extra turn per agent message | [`creel-carry-agent-mail`](design-creel-carry-agent-mail.md) | 2d<br>39m | creel |
| E28 |  | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 3d<br>6h | dotnet |
| E32 |  | **Move rules about lyx behaviour out of this repo's CLAUDE.md into lyx code, doc.go and stencils** | [`claude-md-lyx-rules-out`](design-claude-md-lyx-rules-out.md) | 3d<br>19h | stencil, board, loom |
| E35 |  | **A landing fixer escalation goes to the arbiter before the orch**<br>• Merge-In and Finalize fixer escalate reports reach the arbiter first | [`landing-escalate-arbiter`](design-landing-escalate-arbiter.md) | 1d<br>39m | shed, loom |

### Features (14)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>9d | burler |
| F2 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w<br>1d | ide, fabric |
| F3 |  | **A dobby overview window per beam: alive, runs driven, tasks waiting and why**<br>• Every beam gets a reed window showing whether its dobby lives, what it drives and what each waiting task waits on | [`dobby-overview-window`](design-dobby-overview-window.md) | 1m<br>1m | reed, batten, workspaces |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>2w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w<br>2w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w<br>21h | board, ide, orch |
| F12 |  | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>12d | reed, shuttle, config |
| F13 | H | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>39m | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 10d<br>1d | reed, config, batten |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 10d<br>39m | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 9d<br>1d | burler, shed, orch, quarry, report |
| F18 |  | **Regression as three Drift rows: Drift-Baseline after Discussion, Drift-Await before Webster, Drift-Verify after the Code perch**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 8d<br>21m | shed, webster, batten, config, dotnet |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 2d<br>39m | loom, shed, batten, workspaces, burler |
