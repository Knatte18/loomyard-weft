# Board

Rendered 2026-10-23 12:40 UTC; every age below is as of then.
Prime owner: `osl-1046`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Run tail: Merge-In with alignment, Code-Perch, a PR stub and one locked Land block, for loom and darn**<br>• Loom and darn share one tail after the code: Merge-In with alignment and a thorough fixer, Code-Perch, PR block behind a flag, Finalize re-merging under the parent lock | [`loom-tail-review`](design-loom-tail-review.md) | loom | Plan-Burler-Fix | osl-1046 | 18h<br>4h | loom, burler, shed, fabric, batten |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Each test runs once per tree: disjoint tiers, row entry gates, carried green runs, tmux in final**<br>• Tiers tagged by name, shed row entry gates, verify skipped on an already green tree, tmux in the final gate, gate time measured<br>• `profile: light` | [`gate-runs-once`](design-gate-runs-once.md) | loom | `loom-tail-review` | 6d<br>0m | gate, shed, loom, webster, fabric, batten |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| A1 |  | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 8d<br>8d | reed |
| A2 |  | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 4d<br>4d |  |
| A3 |  | **Audit the twill and lens review setup: worth per perch, recurring finding classes, writer quality**<br>• One audit over the review records once about ten loom runs with the current setup land on Neve | [`review-setup-audit`](design-review-setup-audit.md) | 7d<br>23h | burler, loom, report, stencil, shed |
| A4 |  | **Audit: ideas from tree-sitter-analyzer for quarry and its use in lyx**<br>• What the Python tree-sitter-analyzer offers that quarry or lyx could use if built in Go, what already exists, and what would not pay off | [`tsa-quarry-audit`](design-tsa-quarry-audit.md) | 4d<br>4d | quarry, webster, burler, loom |
| A5 |  | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 3d<br>3d |  |
| A6 |  | **Lens overlap audit: which lenses find the same defects, and a smaller set**<br>• twill-plan 8 to 5 lenses recommended; twill-discussion 7 to 4 and review-code 4 to 3 at low confidence | [`lens-overlap-audit`](design-lens-overlap-audit.md) | 23h<br>23h | burler, loom |
| A7 |  | **Trial: batcher profile wide (budget 300k)**<br>• Hub runs a new batcher profile wide, budget 300k; check cost and fork quality before taking it into the template | [`batch-budget-300k-trial`](design-batch-budget-300k-trial.md) | 22h<br>22h | webster |

### Bugs

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **Discussion check refuses a declared PATTERN text over the overview line cap**<br>• A changes or adds text past pattern.MaxEntryLineChars passes Discussion and first fails at Plan-Write | [`discussion-pattern-line-cap`](design-discussion-pattern-line-cap.md) | 1h<br>1h | loom |
| B2 |  | **Seeded VS Code title-bar colours do not show in task worktrees**<br>• The seeder writes a palette colour, but loom-tail-review settings.json has none and no colour shows | [`ide-title-colours-missing`](design-ide-title-colours-missing.md) | 1h<br>1h | ide |

### Enhancements

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 | L | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 2w<br>2d | fabric |
| E2 |  | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 2w<br>8d | shuttle, codex |
| E3 | L | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w<br> | ide, fabric |
| E4 |  | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w<br>2w | fabric, loom |
| E5 | L | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 2w<br>7d | shed |
| E6 |  | **[.NET 005] Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 2w<br>2d | loom, webster, config, dotnet |
| E7 |  | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 8d<br>8d | shed, loom |
| E8 |  | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 8d<br>8d | shed, batten, loom, config |
| E9 |  | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 11d<br>7d | burler, stencil |
| E10 |  | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 8d<br>8d | board, loom |
| E11 |  | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 8d<br>8d | batten |
| E12 |  | **Git test fixtures trip Cortex XDR on the work laptop**<br>• Parallel git-heavy tests create so many files at once that Cortex XDR killed VS Code and then crashed itself, twice; tests must stay usable on a machine with endpoint protection | [`xdr-git-test-file-burst`](design-xdr-git-test-file-burst.md) | 6d<br>2d | fabric |
| E13 |  | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 7d<br>1d | stencil, shuttle |
| E14 |  | **Speed loom profile: Discussion, Plan and Webster, with review only of the end product**<br>• A loom profile without the Discussion, Plan and Webster perches, reviewing only before landing | [`speed-loom-profile`](design-speed-loom-profile.md) | 20h<br>20h | loom, burler, config |
| E15 |  | **Plan writer cost: sub-planners per part, and twill versus a solo writer**<br>• The Plan writer is 18% of spend; sub-planners per part for large bundles, and a twill versus solo Opus high comparison on cost, rounds and time | [`plan-writer-context`](design-plan-writer-context.md) | 20h<br>20h | loom, stencil |
| E16 |  | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 5d<br>5d | webster, fabric, loom |
| E17 | L | **[WS 007] An anchor per umbrella**<br>• Let an umbrella work in its own subdirectory of the repository, inherited by its tasks; hard to change now | [`umbrella-anchor`](design-umbrella-anchor.md) | 4d<br>4d | fabric, config, workspaces |
| E18 | H | **[WS 005] One dobby daemon per beam owns and drives its runs**<br>• Each beam (a worktree with an orch thread) gets a Go daemon, started by lyx orch start, that follows its runs status files, starts ready tasks, lands, settles mechanical stops and tells the thread only what needs judgement | [`umbrella-daemon`](design-umbrella-daemon.md) | 11d<br>3d | batten, orch, shed, workspaces, loom, board |
| E19 |  | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 4d<br>4d | fabric, batten |
| E20 |  | **Plan and Discussion fixers on Sonnet, once findings are concrete enough**<br>• Webster and Merge fixers are already Sonnet; Plan and Discussion fixers need concrete findings first | [`fixers-on-sonnet`](design-fixers-on-sonnet.md) | 20h<br>20h | burler, claude |
| E21 |  | **The orch Go driver spawns the board-tidy agent after each landing** | [`board-tidy-after-landing`](design-board-tidy-after-landing.md) | 4h<br>4h | orch, board, reed |
| E22 |  | **lyx batten run starts in its own window by default**<br>• Every batten run is started with --window, so the window should be the default and foreground the opt-in. | [`batten-window-default`](design-batten-window-default.md) | 4d<br>4d | batten, reed, orch |
| E23 |  | **The seeded reed attach task in tasks.json: drop it, or bind it to a shortcut**<br>• tasks.json holds one on-demand reed attach task with no shortcut | [`ide-attach-task`](design-ide-attach-task.md) | 1h<br>1h | ide |
| E24 |  | **Every lens fork report is kept verbatim in the run records**<br>• Save each lens fork report in a support file beside lens-review.md, so audits can read what each lens found | [`lens-fork-reports-kept`](design-lens-fork-reports-kept.md) | 23h<br>23h | burler, loom |
| E25 |  | **Ubuntu 24.04 as a stated and checked floor: a PATTERN entry and a mechanical check**<br>• A Neve-written reed change broke on tmux 3.4; nothing states or checks the Ubuntu 24.04 floor | [`ubuntu-2404-floor`](design-ubuntu-2404-floor.md) | 3h<br>3h | reed, gate |
| E26 |  | **A status-bar button turns automatic terminal windows for runs on or off** | [`open-window-toggle`](design-open-window-toggle.md) | 1d<br>1d | batten, reed |
| E27 | L | **Carry agent mail into the pane in full instead of a reminder and a read turn**<br>• A reminder plus lyx creel read costs the receiver an extra turn per agent message | [`creel-carry-agent-mail`](design-creel-carry-agent-mail.md) | 3h<br>3h | creel |
| E28 |  | **[.NET] A small public NUnit repo as sandbox hub for end-to-end C# runs** | [`csharp-sandbox-hub`](design-csharp-sandbox-hub.md) | 1d<br>23h | dotnet |
| E30 |  | **Discussion scope check moves from the alignment agent into the perch lenses, with an escalating finding class** | [`discussion-alignment-into-lenses`](design-discussion-alignment-into-lenses.md) | 1d<br>1d | loom, burler, shed |
| E31 |  | **A row before Discussion aligns the task text against the board, peers and landed work, and adjusts it** | [`entry-align-before-discussion`](design-entry-align-before-discussion.md) | 1d<br>1d | loom, batten, board |
| E32 |  | **Move rules about lyx behaviour out of this repo's CLAUDE.md into lyx code, doc.go and stencils** | [`claude-md-lyx-rules-out`](design-claude-md-lyx-rules-out.md) | 1d<br>1d | stencil, board, loom |

### Features

| # | P | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 2w<br>7d | burler |
| F2 | H | **[WS 006] Umbrella tasks: project threads that spawn and land from their own worktree**<br>• A larger project is an umbrella task with a long-lived pair and its own orch thread; tasks name it as parent, spawn from and land into its branch, and report to its thread; the prime is the root umbrella | [`orch-any-worktree`](design-orch-any-worktree.md) | 2w<br>1d | fabric, batten, orch, board, reed, creel, shed, workspaces |
| F3 |  | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 2w<br> | quarry |
| F4 |  | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 2w<br> | stencil |
| F5 |  | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 2w<br>2w | fabric |
| F6 |  | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 2w<br> | reed |
| F7 |  | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 2w<br> | loom |
| F8 |  | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w<br>2w | fabric |
| F9 |  | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 2w<br>2w | config, board |
| F10 |  | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 2w<br> | shuttle |
| F11 |  | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w<br>2w | board, ide, orch |
| F12 |  | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 2w<br>10d | reed, shuttle, config |
| F13 | H | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 12d<br>4d | webster |
| F14 | L | **[.NET 003] Scoped build and test for C# hubs**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 11d<br>1d | shed, webster, dotnet |
| F15 |  | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 8d<br>8d | reed, config |
| F16 | H | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 8d<br>3h | creel, shuttle, loom, stencil, report |
| F17 |  | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 7d<br>6d | burler, shed, orch, quarry, report |
| F18 |  | **[.NET 004] Asynchronous test tiers in shed: the numerical baseline captured before implementation, compared at the final gate**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 6d<br>2d | shed, webster, batten, config, dotnet |
| F19 | H | **[WS 004] Board projects and pairs created from a parent pair**<br>• A task names its project; a project groups tasks in order and may have its own worktree; fabric and batten create a project task pair from that branch, batten verbs run from any worktree | [`pair-from-parent`](design-pair-from-parent.md) | 4d<br>4d | fabric, batten, board, workspaces |
| F20 |  | **Large tasks: Discussion, a pre-planner, and the pieces as darn runs inside the task's beam**<br>• Discussion decides scope, a pre-planner splits it into contracted pieces, each piece runs as darn from the task's beam, a whole-task perch reviews, then align and land | [`loom-beam-darns`](design-loom-beam-darns.md) | 19h<br>1m | loom, shed, batten, workspaces, burler |
