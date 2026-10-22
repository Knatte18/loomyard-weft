# Board

Rendered 2026-10-22 06:24 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### High priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **One machine takes over from another: owner machines, lyx adopt, landing on the parent owner**<br>• Owner machine per entry, lyx adopt of named entries, landing only on the parent owner, owner outcome in the origin reconcile, Open-Window per machine | [`machine-handover`](design-machine-handover.md) | loom | Discussion-Bouncer | 2d | 2d | batten, board, orch, fabric |
| 2 | **[.NET 006] LoomYard uses quarry C# glyphs: version raise, alphabet by extension**<br>• LoomYard side of C# glyphs, after the quarry hub csharp-alphabet lands and is tagged<br>• `profile: light` | [`quarry-csharp`](design-quarry-csharp.md) | loom | Discussion-Burler-Review | 1d | 24m | quarry, dotnet |

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 3 | **lyx loom decision add notifies the run live LLM producer of the new decision**<br>• With the Go driver no agent lives for the whole run, so decision add tells the working LLM producer if any, and later producers and PR-Gate catch the rest<br>• `darn: review_hard_max_bounces 2`<br>• `darn: review_max_bounces 1` | [`decision-add-notifies`](design-decision-add-notifies.md) | darn | Merge-In | 3d | 18m | loom, creel |
| 4 | **Darn-Write stops looping on verify failures it cannot own, and the flaky reed boot test behind them**<br>• A verify failure in code the task did not touch, or one the parent already fixed, no longer loops through Darn-Write; the reed boot race is made deterministic and the impacted set stops picking unreachable packages<br>• `profile: light` | [`darn-verify-fixes`](design-darn-verify-fixes.md) | loom | Discussion-Burler-Fix | 4d | 34m | batten, gate, loom, shed, reed |
| 5 | **lyx gate test --upto N with manual tiers, and a lyx gate status verb**<br>• --upto N runs tiers 0..N skipping manual ones; gate status shows who holds each gate slot and since when<br>• `darn: review_hard_max_bounces 2`<br>• `darn: review_max_bounces 1` | [`gate-tier-verbs`](design-gate-tier-verbs.md) | darn | Darn-Burler-Fix | 19h | 14m | gate |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Remove the Webster kick-start pack**<br>• Drop every kick-start pack layout and its code; glyphs stay in the plan and its gates<br>• `darn: review_hard_max_bounces 2`<br>• `darn: review_max_bounces 1` | [`kickstart-pack-remove`](design-kickstart-pack-remove.md) | darn | `quarry-csharp` | 4d | 15m | webster, quarry, loom |
| 2 | **tender pause does not hold a verify gate's re-prompt, and creel interrupt is refused under the pause**<br>• `profile: light` | [`tender-pause-holds-reprompt`](design-tender-pause-holds-reprompt.md) | loom | `darn-verify-fixes` | 1d | 32m | creel, shed, webster |
| 3 | **Finalize blocks on 'worktree dirty' without naming the tree or paths, and the loom's own run record counts as dirty**<br>• `profile: light` | [`finalize-dirty-names-paths`](design-finalize-dirty-names-paths.md) | loom | `machine-handover`, `darn-verify-fixes` | 1d | 1d | fabric, loom |

### Layer B

Starts when every entry it names under After is done.

#### High priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Every process a run spawns has one owner that ends them all**<br>• Owner records for run processes; pause, reed down, fabric remove and teardown end the whole group; tender status shows orphans; a hub pause reaches background work; a finished reviewer never waits on its background shells | [`run-process-ownership`](design-run-process-ownership.md) | loom | `machine-handover`, `tender-pause-holds-reprompt` | 7d | 43m | batten, burler, fabric, loom, shed, shuttle, webster |

### Layer C

Starts when every entry it names under After is done.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **One batten verb ends a run for good, live or stopped, before or after its pair exists**<br>• Abandon or abort a batten run in any state: end its processes and strands, clear the claim, remove records and scratch, optionally the pair; batten refuses a dirty source up front<br>• `profile: light` | [`batten-cancel-verb`](design-batten-cancel-verb.md) | loom | `run-process-ownership`, `darn-verify-fixes` | 1d | 43m | batten, reed, shed |
| 2 | **Orch lifecycle: dead strands count as dead, the watcher restarts, cycle requests need no reply and can be withdrawn**<br>• Strand liveness checks the engine process and placement, a dead orch watcher restarts, the note write is seen without a reply, and a pending distill/refresh can be withdrawn<br>• `profile: light` | [`orch-strand-cycle`](design-orch-strand-cycle.md) | loom | `machine-handover`, `darn-verify-fixes`, `run-process-ownership` | 3d | 18h | orch, reed |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 6d | 6d | reed |
| 2 | **Classify Plan lens and review findings to make the Plan writer get it right first**<br>• Find what Plan lenses and Plan-Burler-Review keep catching, and stop each recurring class at the writer | [`plan-writer-finding-classes`](design-plan-writer-finding-classes.md) | 6d | 5d | loom, burler, stencil, shed |
| 3 | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 3d | 3d |  |
| 4 | **Audit the use of the twill and lens review setup at every perch**<br>• After about ten loom runs, measure what the lens fans and twill fixers catch per perch against their cost, and set the standard review rounds from it | [`review-setup-audit`](design-review-setup-audit.md) | 3d | 3d | burler, loom, report |
| 5 | **Audit: ideas from tree-sitter-analyzer for quarry and its use in lyx**<br>• What the Python tree-sitter-analyzer offers that quarry or lyx could use if built in Go, what already exists, and what would not pay off | [`tsa-quarry-audit`](design-tsa-quarry-audit.md) | 3d | 3d | quarry, webster, burler, loom |
| 6 | **Audit: Discussion and Plan quality before and after the twill writers**<br>• Plan round-1 findings fell about half with the twill Plan writer; Discussion is the same or slightly better; fork0 and the code list added no further gain | [`twill-writer-quality-audit`](design-twill-writer-quality-audit.md) | 3d | 3d | loom, burler, stencil |
| 7 | **Sweep go_parallel on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 2d | 1d |  |
| 8 | **[.NET 001] Measure NORCE Models build and test on OSL-1046 with Cortex XDR**<br>• Clock restore, build and test of Models cold, warm and in a second checkout before building .NET support | [`dotnet-bench-osl1046`](design-dotnet-bench-osl1046.md) | 1d | 1d | dotnet |

### Bugs

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **fabric add refuses an existing local branch instead of adopting it**<br>• lyx fabric add adopts origin branches but refuses a local unmarked warp branch, so main or an umbrella branch cannot get a pair | [`fabric-add-adopt-local`](design-fabric-add-adopt-local.md) | 23h | 23h | fabric, workspaces |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 005] One dobby daemon per beam owns and drives its runs**<br>• Each beam (a worktree with an orch thread) gets a Go daemon, started by lyx orch start, that follows its runs status files, starts ready tasks, lands, settles mechanical stops and tells the thread only what needs judgement | [`umbrella-daemon`](design-umbrella-daemon.md) | 9d | 2d | batten, orch, shed, workspaces, loom, board |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 2w | 6d | shuttle, codex |
| 3 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w | 2w | fabric, loom |
| 4 | **[.NET 005] Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 2w | 19h | loom, webster, config, dotnet |
| 5 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 7d | 7d | shed, loom |
| 6 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 6d | 6d | shed, batten, loom, config |
| 7 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 10d | 5d | burler, stencil |
| 8 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 6d | 6d | board, loom |
| 9 | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 6d | 6d | batten |
| 10 | **Git test fixtures trip Cortex XDR on the work laptop**<br>• Parallel git-heavy tests create so many files at once that Cortex XDR killed VS Code and then crashed itself, twice; tests must stay usable on a machine with endpoint protection | [`xdr-git-test-file-burst`](design-xdr-git-test-file-burst.md) | 5d | 1d | fabric |
| 11 | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 5d | 4d | stencil, shuttle |
| 12 | **Entry gate on shed rows: a check decides whether a row runs at all**<br>• Replace darn recipe edge rewrite and the switched producer flag check with one shed entry gate per row; false means Done without acting. | [`row-entry-gate`](design-row-entry-gate.md) | 4d | 3d | shed, loom, batten |
| 13 | **Use quarry Enclose for mechanical checks: verify failures to plan cards**<br>• Map tool-output file:line to member glyphs, then to the card and batch that changed them | [`enclose-checks`](design-enclose-checks.md) | 2w | 4d | quarry, webster, loom |
| 14 | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 3d | 3d | webster, fabric, loom |
| 15 | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 3d | 3d | fabric, batten |
| 16 | **Plan gate does not catch declared PATTERN.md growth that exceeds the headroom line** | [`plan-gate-pattern-headroom`](design-plan-gate-pattern-headroom.md) | 3d | 3d | loom, burler |
| 17 | **Plan cards mark additive edits so shared spots stop chaining the card DAG**<br>• An Extends label for edits that only add to a ref; two Extends on one ref are no dependency; measure with plandag first | [`plan-extends-label`](design-plan-extends-label.md) | 3d | 3d | loom, webster, burler |
| 18 | **lyx batten run starts in its own window by default**<br>• Every batten run is started with --window, so the window should be the default and foreground the opt-in. | [`batten-window-default`](design-batten-window-default.md) | 2d | 2d | batten, reed, orch |
| 19 | **A halt a plan card asks for reaches the parent as a question, not as a batch failure**<br>• A card-requested halt went through recover-batch, tackler and arbiter as a failure; it should go straight to the parent and the next attempt should carry the decision | [`card-halt-is-question`](design-card-halt-is-question.md) | 2d | 2d | webster, shed, loom |
| 20 | **Cortex XDR holds stack up across a loom run's card, batch, round and whole-suite gates**<br>• Each gate that tests a just-changed package waits 8-10 s for XDR; a 10-15 card run pays that many times over | [`loom-gates-xdr-stacking`](design-loom-gates-xdr-stacking.md) | 1d | 1d | webster, loom, gate |
| 21 | **[.NET 007] Language-specific lens sets seeded per hub**<br>• Every hub seeds lenses.yaml from one Go-repo template; seed by the hub language instead | [`lenses-language-sets`](design-lenses-language-sets.md) | 1d | 1d | burler, config, dotnet |
| 22 | **Revisit late review rounds after the twill-plan lens change**<br>• After 4-5 loom runs past 504c5552, check whether discussion-review round 3 and plan-review round 2 still find anything | [`review-rounds-after-twill-plan`](design-review-rounds-after-twill-plan.md) | 13h | 13h | burler, loom |
| 23 | **Test runs repeat at the end of a loom run: tier0 inside every tier, the suite four times, tmux only at Publish**<br>• Disjoint tiers, a green run that carries forward, tmux in more gates, and a faster fabricengine suite; with the 2026-10-22 tier timings | [`gate-test-repeats`](design-gate-test-repeats.md) | 36m | 36m | gate, loom, webster, fabric |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 24 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 2w | 22h | fabric |
| 25 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w |  | ide, fabric |
| 26 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 2w | 6d | shed |
| 27 | **[WS 007] An anchor per umbrella**<br>• Let an umbrella work in its own subdirectory of the repository, inherited by its tasks; hard to change now | [`umbrella-anchor`](design-umbrella-anchor.md) | 3d | 3d | fabric, config, workspaces |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 006] Umbrella tasks: project threads that spawn and land from their own worktree**<br>• A larger project is an umbrella task with a long-lived pair and its own orch thread; tasks name it as parent, spawn from and land into its branch, and report to its thread; the prime is the root umbrella | [`orch-any-worktree`](design-orch-any-worktree.md) | 2w | 2d | fabric, batten, orch, board, reed, creel, shed, workspaces |
| 2 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 10d | 3d | webster |
| 3 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 6d | 3d | creel, shuttle, loom, stencil, report |
| 4 | **[WS 004] Board projects and pairs created from a parent pair**<br>• A task names its project; a project groups tasks in order and may have its own worktree; fabric and batten create a project task pair from that branch, batten verbs run from any worktree | [`pair-from-parent`](design-pair-from-parent.md) | 3d | 3d | fabric, batten, board, workspaces |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 2w | 5d | burler |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 2w |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 2w |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 2w | 2w | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 2w |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 2w |  | loom |
| 11 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w | 2w | fabric |
| 12 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 2w | 13d | config, board |
| 13 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 2w |  | shuttle |
| 14 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w | 2w | board, ide, orch |
| 15 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 13d | 9d | reed, shuttle, config |
| 16 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 6d | 6d | reed, config |
| 17 | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5d | 4d | burler, shed, orch, quarry, report |
| 18 | **[.NET 004] Asynchronous test tiers in shed: the numerical baseline captured before implementation, compared at the final gate**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 4d | 19h | shed, webster, batten, config, dotnet |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **[.NET 003] Scoped build and test for C# hubs**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 9d | 1d | shed, webster, dotnet |
