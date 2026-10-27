# Board

Rendered 2026-10-27 13:24 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Projects (2)

Groups of entries; a beam spawns its members from its own worktree.

### P1 Workspaces: beams, pairs from a parent pair, dobby

Beams that develop a project in their own worktree, driven by a dobby daemon

grouping

_None._

### P2 .NET: LoomYard for C# repositories

Running LoomYard on C#/.NET repositories such as Models: scoped gates, regression, a sandbox hub

grouping

| # | Member | Kind | Standing |
| --- | --- | --- | --- |
| E28 | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | Note | Note |

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Bench Codex review block v2 on P1: one high run against two medium**<br>• Forked Astra fan with block v2 and a 20-call fork cap on P1: 1 high and 2 medium runs<br>• `profile: docs` | [`codex-v2-p1-bench`](design-codex-v2-p1-bench.md) | darn | Darn-Write | neve | 0m<br>0m | codex, burler |
| T2 | **Bench Codex review block v2 on P2: one high run against two medium**<br>• Forked Astra fan with block v2 and a 20-call fork cap on P2: 1 high and 2 medium runs<br>• `profile: docs` | [`codex-v2-p2-bench`](design-codex-v2-p2-bench.md) | darn | Worktree-Create | neve | 0m<br>0m | codex, burler |
| T3 | **Bench Codex review block v2 on P4: one high run against two medium**<br>• Forked Astra fan with block v2 and a 20-call fork cap on P4: 1 high and 2 medium runs<br>• `profile: docs` | [`codex-v2-p4-bench`](design-codex-v2-p4-bench.md) | darn | Darn-Write | neve | 0m<br>0m | codex, burler |
| T4 | **Study what the Claude lens fan misses that Codex finds, and how to tune it**<br>• Desk study over the Astra benches: classify Claude-missed real findings and propose stencil changes<br>• `profile: docs` | [`claude-review-gaps`](design-claude-review-gaps.md) | darn | Worktree-Create | neve | 0m<br>0m | burler, stencil, claude |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (47)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs (7)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Find why a running run shows a dead waiter in the activity reader** | [`idle-notice-dead-waiter-cause`](design-idle-notice-dead-waiter-cause.md) | 3d<br>3d | batten, shuttle |
| B2 |  | **A read creates directories: state.ReadJSON mkdirs, so a hub reader leaves a real _lym in a half-built pair** | [`hub-reader-creates-lym`](design-hub-reader-creates-lym.md) | 1d<br>3h | fabric, orch, batten |
| B3 |  | **Dobby dies at a deploy instead of re-executing onto the new binary**<br>• After a landing deploy the prime dobby was gone, its board-tidy agent left open | [`dobby-reexec-on-deploy`](design-dobby-reexec-on-deploy.md) | 21h<br>3h | workspaces, batten |
| B4 |  | **Board README lists draft tasks under Ready** | [`readme-draft-not-ready`](design-readme-draft-not-ready.md) | 1d<br>1d | board |
| B5 |  | **Board tidy counts landings per machine, not per owner**<br>• A dobby that does not own root tidies landings another machine already tidied | [`tidy-count-per-owner`](design-tidy-count-per-owner.md) | 6h<br>6h | workspaces |
| B6 |  | **A gate test writes trace logs into the live hub and Bolt commits them**<br>• trace-*.log from a run gate test landed in _board/.lym/logs and board sync committed them | [`test-logs-leak-into-bolt`](design-test-logs-leak-into-bolt.md) | 3h<br>3h | gate, board, testkit |
| B7 |  | **An instruction typed into a pane is lost when the tackler re-spawns the row**<br>• A re-spawned row session starts from its prompt alone; operator input to the old session is dropped | [`pane-input-lost-on-respawn`](design-pane-input-lost-on-respawn.md) | 3h<br>3h | shed, shuttle, reed |

### Enhancements (20)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Beam promote keeps the Discussion record as part of the beam**<br>• When Discussion finds a task should be a project, promote carries its decision record into the beam instead of dropping it | [`beam-promote-keeps-discussion`](design-beam-promote-keeps-discussion.md) | 2d<br>3h | workspaces, batten, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 3w<br>13m | shuttle, codex |
| E3 |  | **Dobby log: changes and a heartbeat only, and bounded**<br>• dobby.log grows without bound with a line per pass; log changes plus a 10-minute heartbeat, under retention | [`dobby-log-bounded`](design-dobby-log-bounded.md) | 21h<br>21h | workspaces |
| E5 |  | **Decide the layout of projects in the board README**<br>• The operator is not satisfied with how the README renders projects and their members; the layout is to be decided | [`board-projects-layout`](design-board-projects-layout.md) | 2d<br>2d | board |
| E6 |  | **Board README: one rendered file per project, research notes first**<br>• Each project renders its own file linked from its parent; README.md is the root project; research notes come right after tasks | [`readme-per-project-files`](design-readme-per-project-files.md) | 1d<br>1d | board |
| E7 |  | **Run verbs in one shared namespace, not under lym loom**<br>• lym loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 12d<br>3h | shed, loom |
| E8 |  | **Darn runs Drift-Await after Drift-Baseline instead of a waiting capture** | [`darn-drift-await-row`](design-darn-drift-await-row.md) | 1d<br>1d | shed, loom |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 2w<br>3h | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 12d<br>12d | board, loom |
| E11 |  | **Loom and darn Finalize resolve a moved parent themselves instead of rerouting** | [`loom-finalize-merges-itself`](design-loom-finalize-merges-itself.md) | 1d<br>22h | loom, batten |
| E12 |  | **Adopt machine check: a gate key of its own, and visible while it runs**<br>• Adopt runs its machine check silently in the background under the finalize gate key | [`adopt-check-visible`](design-adopt-check-visible.md) | 1d<br>3h | gate, board |
| E13 |  | **VS Code colours by project and worktree, and lym ide seed with no slug**<br>• Title bar shows the project (prime or beam), the activity bar the worktree; text colour by luminance; ide seed with no slug seeds the current worktree | [`ide-seed-current-worktree`](design-ide-seed-current-worktree.md) | 4h<br>3h | ide, workspaces |
| E14 |  | **No test is skipped by design**<br>• Windows-only tests count as skips on Linux, and TestBuildLoomShed_OutputShape skips on every machine | [`test-skips-os-tags`](design-test-skips-os-tags.md) | 1d<br>1d | testkit, reed, loom, fabric, gate |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 9d<br>3h | webster, fabric, loom |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 8d<br>3h | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 4d<br>4d | burler, claude |
| E27 | L | **Creel delivers agent mail in full at the receiver's next tool boundary**<br>• A reminder plus lym creel read costs the receiver an extra turn per agent message | [`creel-carry-agent-mail`](design-creel-carry-agent-mail.md) | 4d<br>3h | creel |
| E28 |  | **A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 5d<br>3h | dotnet |
| E32 |  | **Move rules about lym behaviour out of this repo's CLAUDE.md into lym code, doc.go and stencils** | [`claude-md-lym-rules-out`](design-claude-md-lym-rules-out.md) | 5d<br>3h | stencil, board, loom |
| E35 |  | **A landing fixer escalation goes to the arbiter before the orch**<br>• Merge-In and Finalize fixer escalate reports reach the arbiter first | [`landing-escalate-arbiter`](design-landing-escalate-arbiter.md) | 3d<br>22h | shed, loom |

### Features (13)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>3h | burler |
| F2 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lym-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 3w<br>3h | ide, fabric |
| F3 |  | **One verb that pulls the whole hub, Bolt included, while the orch runs**<br>• Pull the prime pair, Bolt and local beams in one safe verb | [`hub-pull-verb`](design-hub-pull-verb.md) | 3h<br>3h | fabric, board |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 3w<br>3w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 3w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 3w<br>3w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 3w<br>3h | board, ide, orch |
| F12 |  | **Spawn audit log: every process lym starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>3h | reed, shuttle, config |
| F13 | H | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 2w<br>3h | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 12d<br>3h | reed, config, batten |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 12d<br>3h | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 11d<br>3h | burler, shed, orch, quarry, report |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 4d<br>3h | loom, shed, batten, workspaces, burler |

### Researchs (7)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 8d<br>1d |  |
| R2 |  | **Audit the twill and lens review setup: worth per perch, recurring finding classes, writer quality**<br>• One audit over the review records once about ten loom runs with the current setup land on Neve | [`review-setup-audit`](design-review-setup-audit.md) | 11d<br>3h | burler, loom, report, stencil, shed |
| R3 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 7d<br>1d |  |
| R4 |  | **Lens overlap audit: which lenses find the same defects, and a smaller set**<br>• twill-plan 8 to 5 lenses recommended; twill-discussion 7 to 4 and review-code 4 to 3 at low confidence | [`lens-overlap-audit`](design-lens-overlap-audit.md) | 5d<br>3h | burler, loom |
| R5 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 4d<br>3h | webster |
| R6 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 4d<br>3h | loom, stencil |
| R8 |  | **Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures**<br>• About 30 ideas judged against our stencils and rules; top 8 ranked for a bench, a few cheap enough without one | [`claude-md-thread-ideas`](design-claude-md-thread-ideas.md) | 22h<br>3h | stencil, loom, burler |
