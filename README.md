# Board

Rendered 2026-10-24 14:19 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (6)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Clean up PATTERN.md: far too many entries**<br>• `profile: light` | [`pattern-cleanup`](design-pattern-cleanup.md) | darn | Describe | neve | 6h<br>3h | stencil |

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Card gate (--tier 0) does not vet a card's packages under the other tier tag sets, so helper dependencies across tagged files fail only in the batch gate**<br>• `profile: light` | [`card-gate-vets-all-tag-sets`](design-card-gate-vets-all-tag-sets.md) | darn |  | 41m<br>32m | gate, webster |

### Layer A (4)

Waits only on Running or Ready entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Run profiles as one ladder: speed, light, solid, wide**<br>• Replace budget, light, docs and darn-wide with one profile ladder shared by darn and loom<br>• `profile: light` | [`run-profiles-ladder`](design-run-profiles-ladder.md) | darn | `pattern-cleanup` | 1d<br>3h | config, loom, burler |
| T4 | **[.NET] Scoped build and test for C# hubs, and the Models verification wants**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | loom | `pattern-cleanup`, `card-gate-vets-all-tag-sets` | 2w<br>4m | shed, webster, loom, config, dotnet |
| T6 | **Fork git-mutation guard fails a run on git commands confined to a scratch repo**<br>• `profile: light` | [`fork-git-guard-scratch-repo`](design-fork-git-guard-scratch-repo.md) | darn | `pattern-cleanup` | 21h<br>3h | shed, burler |
| T9 | **Umbrella groundwork: project entries, pairs from a parent pair, and umbrella tasks that spawn and land from their own worktree**<br>• [WS 004+006] Board projects, fabric pairs from a parent pair, batten and orch from any worktree, umbrella tasks | [`umbrella-groundwork`](design-umbrella-groundwork.md) | loom | `pattern-cleanup` | 3w<br>3h | fabric, batten, orch, board, reed, creel, shed, workspaces |

## Notes (45)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits (6)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| A2 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 6d<br>6d |  |
| A3 |  | **Audit the twill and lens review setup: worth per perch, recurring finding classes, writer quality**<br>• One audit over the review records once about ten loom runs with the current setup land on Neve | [`review-setup-audit`](design-review-setup-audit.md) | 8d<br>2d | burler, loom, report, stencil, shed |
| A5 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 4d<br>4d |  |
| A6 |  | **Lens overlap audit: which lenses find the same defects, and a smaller set**<br>• twill-plan 8 to 5 lenses recommended; twill-discussion 7 to 4 and review-code 4 to 3 at low confidence | [`lens-overlap-audit`](design-lens-overlap-audit.md) | 2d<br>6h | burler, loom |
| A7 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 2d<br>5h | webster |
| A8 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 1d<br>4h | loom, stencil |

### Bugs (3)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Find why a running run shows a dead waiter in the activity reader** | [`idle-notice-dead-waiter-cause`](design-idle-notice-dead-waiter-cause.md) | 5h<br>5h | batten, shuttle |
| B2 |  | **A run never files a task, at most a note**<br>• gate-runs-once filed a follow-up task (default loom) that then auto-started; runs may file only notes | [`runs-never-write-board`](design-runs-never-write-board.md) | 55m<br>53m | board, loom, stencil |
| B7 |  | **Webster batch whose card verify needs an orch-only change dead-ends: recover-batch ends dead with 'asking', then the run halts repeatedly with growing history** | [`webster-orch-need-to-parent`](design-webster-orch-need-to-parent.md) | 41m<br>41m | webster, loom |

### Enhancements (23)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Plan conflict from a moved tree: a small fixer before the Plan perch**<br>• A tree_moved plan conflict naming exact glyphs goes to a small card fixer, not a full Plan review round | [`plan-conflict-small-fixer`](design-plan-conflict-small-fixer.md) | 4h<br>4h | webster, loom |
| E2 |  | **Codex CLI as a second shuttle engine: codexengine**<br>• codexengine for single-session roles, built from the 2026-10-24 spike report and claudeengine list of needs | [`codex-engine`](design-codex-engine.md) | 2w<br>1h | shuttle, codex |
| E3 |  | **Plan check does not project the lint-pattern step per card, so a wrong plan-review fix passes review and fails only at self-check** | [`plan-check-lint-pattern-per-card`](design-plan-check-lint-pattern-per-card.md) | 41m<br>41m | loom, burler |
| E4 |  | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w<br>2w | fabric, loom |
| E7 |  | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 9d<br>9d | shed, loom |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 12d<br>5h | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 9d<br>9d | board, loom |
| E13 |  | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 8d<br>2d | stencil, shuttle |
| E14 |  | **An adopt runs the test suite on the adopting machine**<br>• Catch breaks from a machine change (Ubuntu 26.04, 24.04, Windows 11) at adopt time | [`adopt-runs-tests`](design-adopt-runs-tests.md) | 8h<br>56m | fabric, gate |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 6d<br>6d | webster, fabric, loom |
| E17 | L | **[WS 007] An anchor per umbrella**<br>• Let a beam work in its own subdirectory of the repository, inherited by its tasks; loom-sized, best with or after umbrella-groundwork | [`umbrella-anchor`](design-umbrella-anchor.md) | 6d<br>6h | fabric, config, workspaces |
| E18 | H | **[WS 005] One dobby daemon per beam owns and drives its runs**<br>• Each beam (a worktree with an orch thread) gets a Go daemon, started by lyx orch start, that follows its runs status files, starts ready tasks, lands, settles mechanical stops and tells the thread only what needs judgement | [`umbrella-daemon`](design-umbrella-daemon.md) | 12d<br>5h | batten, orch, shed, workspaces, loom, board |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 5d<br>5d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 1d<br>1d | burler, claude |
| E22 |  | **lyx batten run starts in its own window by default**<br>• Every batten run is started with --window, so the window should be the default and foreground the opt-in. | [`batten-window-default`](design-batten-window-default.md) | 5d<br>5d | batten, reed, orch |
| E25 |  | **Ubuntu 24.04 as a stated and checked floor: a PATTERN entry and a mechanical check**<br>• A Neve-written reed change broke on tmux 3.4; nothing states or checks the Ubuntu 24.04 floor | [`ubuntu-2404-floor`](design-ubuntu-2404-floor.md) | 1d<br>56m | reed, gate |
| E27 | L | **Carry agent mail into the pane in full instead of a reminder and a read turn**<br>• A reminder plus lyx creel read costs the receiver an extra turn per agent message | [`creel-carry-agent-mail`](design-creel-carry-agent-mail.md) | 1d<br>7h | creel |
| E28 |  | **[.NET] A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 2d<br>2d | dotnet |
| E30 |  | **Discussion scope check moves from the alignment agent into the perch lenses, with an escalating finding class** | [`discussion-alignment-into-lenses`](design-discussion-alignment-into-lenses.md) | 2d<br>7h | loom, burler, shed |
| E31 |  | **A row before Discussion aligns the task text against the board, peers and landed work, and adjusts it** | [`entry-align-before-discussion`](design-entry-align-before-discussion.md) | 2d<br>2d | loom, batten, board |
| E32 |  | **Move rules about lyx behaviour out of this repo's CLAUDE.md into lyx code, doc.go and stencils** | [`claude-md-lyx-rules-out`](design-claude-md-lyx-rules-out.md) | 2d<br>2d | stencil, board, loom |
| E34 |  | **Rename the Merge-In row to Parent-Align**<br>• The row aligns the task with what the parent brought in, not only merges | [`merge-in-parent-align`](design-merge-in-parent-align.md) | 7h<br>7h | loom, shed, stencil |
| E35 |  | **A landing fixer escalation goes to the arbiter before the orch**<br>• Merge-In and Finalize fixer escalate reports reach the arbiter first | [`landing-escalate-arbiter`](design-landing-escalate-arbiter.md) | 7h<br>7h | shed, loom |

### Features (13)

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 3w<br>8d | burler |
| F2 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w<br>5h | ide, fabric |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 2w<br>2w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 2w<br> | reed |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w<br>2w | fabric |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w<br>2w | board, ide, orch |
| F12 |  | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>12d | reed, shuttle, config |
| F13 | H | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 13d<br>5d | webster |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 9d<br>5h | reed, config, batten |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 9d<br>2h | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 8d<br>5h | burler, shed, orch, quarry, report |
| F18 |  | **[.NET 004] Asynchronous test tiers in shed: the numerical baseline captured before implementation, compared at the final gate**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 7d<br>56m | shed, webster, batten, config, dotnet |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 1d<br>56m | loom, shed, batten, workspaces, burler |
