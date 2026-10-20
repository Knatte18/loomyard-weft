# Board

Rendered 2026-10-20 06:38 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Arbiter verdict with an unquoted verbs item containing ': ' is unparseable**<br>• `darn: review_hard_max_bounces 1`<br>• `darn: review_max_bounces 1` | [`arbiter-verdict-quoting`](design-arbiter-verdict-quoting.md) | darn | Darn-Write | 13h | 36m | shed, loom |

### Ready

Waits on nothing open; can start now.

#### High priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Origin reconcile before anything that matters, and pushes that never fail silently**<br>• ls-remote over warp and records repos at orch start, wake, batten resume, Merge-In and Finalize; fast-forward, landed-elsewhere cleanup, stop on divergence; rejected push retries and reports | [`origin-reconcile`](design-origin-reconcile.md) | loom |  | 2d | 24m | fabric, batten, orch, creel, shed |
| 2 | **Goto, resume and stops keep what a run already settled**<br>• Goto as a controlled re-entry: converged verdicts, spent review budget and the pause reason survive a goto or stop; no friction session per stop; a zero-fork review round is retried | [`run-reentry`](design-run-reentry.md) | loom |  | 13h | 25m | shed, loom, burler, shuttle |

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 3 | **Lenses and fans move into a lenses module of their own, listed in full in the hub file**<br>• A lenses module holds every lens and fan for reviewers, fixers and twill writers; reconcile keeps the hub file complete; the 7-lens twill-discussion fan becomes the default<br>• `loom: review_hard_max_bounces 3`<br>• `loom: review_max_bounces 2` | [`lenses-module`](design-lenses-module.md) | loom |  | 13h | 13h | burler, loom, config |
| 4 | **Gate builds with -trimpath and verify runs the unit tests once**<br>• -trimpath makes unchanged test binaries byte-identical across worktrees so XDR passes them at once, and verify runs only the integration tier, which includes the unit tests<br>• `darn: review_hard_max_bounces 1`<br>• `darn: review_max_bounces 1` | [`gate-fewer-new-binaries`](design-gate-fewer-new-binaries.md) | darn |  | 9m | 5m | config, shed |

### Layer A

Waits only on Running or Ready entries.

#### High priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **One machine takes over from another: owner machines, lyx adopt, landing on the parent owner**<br>• Owner machine per entry, lyx adopt of named entries, landing only on the parent owner, owner outcome in the origin reconcile, Open-Window per machine | [`machine-handover`](design-machine-handover.md) | loom | `origin-reconcile` | 19h | 15h | batten, board, orch, fabric |
| 2 | **Every process a run spawns has one owner that ends them all**<br>• Owner records for run processes; pause, reed down, fabric remove and teardown end the whole group; tender status shows orphans; batten abandon ends a dead run; a hub pause reaches background work | [`run-process-ownership`](design-run-process-ownership.md) | loom | `run-reentry`, `origin-reconcile` | 5d | 24m | batten, fabric, loom, shed, shuttle, webster |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 4d | 4d | reed |
| 2 | **Classify Plan lens and review findings to make the Plan writer get it right first**<br>• Find what Plan lenses and Plan-Burler-Review keep catching, and stop each recurring class at the writer | [`plan-writer-finding-classes`](design-plan-writer-finding-classes.md) | 4d | 3d | loom, burler, stencil, shed |
| 3 | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 1d | 1d |  |
| 4 | **Audit the use of the twill and lens review setup at every perch**<br>• After about ten loom runs, measure what the lens fans and twill fixers catch per perch against their cost, and set the standard review rounds from it | [`review-setup-audit`](design-review-setup-audit.md) | 1d | 1d | burler, loom, report |
| 5 | **Audit: ideas from tree-sitter-analyzer for quarry and its use in lyx**<br>• What the Python tree-sitter-analyzer offers that quarry or lyx could use if built in Go, what already exists, and what would not pay off | [`tsa-quarry-audit`](design-tsa-quarry-audit.md) | 1d | 1d | quarry, webster, burler, loom |
| 6 | **Audit: Discussion and Plan quality before and after the twill writers**<br>• Plan round-1 findings fell about half with the twill Plan writer; Discussion is the same or slightly better; fork0 and the code list added no further gain | [`twill-writer-quality-audit`](design-twill-writer-quality-audit.md) | 1d | 1d | loom, burler, stencil |
| 7 | **Clock the unit and integration tiers on OSL-1046 with Cortex XDR**<br>• Time both gate tiers on the work laptop on a quiet machine and log them in docs beside the Neve numbers, with full hardware info | [`bench-osl1046-xdr`](design-bench-osl1046-xdr.md) | 22m | 22m |  |

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **webster create-not-done check rejects a Create target the card removes itself** | [`create-target-transient`](design-create-target-transient.md) | 1d | 1d | webster, loom, quarry |
| 2 | **orch status reports a live strand after the orch Claude session exits, so --continue is refused** | [`orch-strand-live-after-exit`](design-orch-strand-live-after-exit.md) | 16h | 16h | orch, reed |
| 3 | **reedengine: TestOpenWindow_RealTmux/ClosesWhenItsCommandExitsAtOnce flakes: window read "on"** | [`openwindow-exit-race`](design-openwindow-exit-race.md) | 0m | 0m | reed |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 005] One dobby daemon per beam owns and drives its runs**<br>• Each beam (a worktree with an orch thread) gets a Go daemon, started by lyx orch start, that follows its runs status files, starts ready tasks, lands, settles mechanical stops and tells the thread only what needs judgement | [`umbrella-daemon`](design-umbrella-daemon.md) | 7d | 14h | batten, orch, shed, workspaces, loom, board |
| 2 | **Align each perch writer twill lenses with its reviewer lenses, Discussion first**<br>• The Discussion writer runs emphasis lenses while its reviewer hunts design holes by procedure, and design holes are what the review still finds | [`writer-reviewer-lens-align`](design-writer-reviewer-lens-align.md) | 1d | 13h | burler, loom, config |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 2w | 4d | shuttle, codex |
| 4 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w | 2w | fabric, loom |
| 5 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 2w | 4d | loom, webster, config |
| 6 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 5d | 5d | shed, loom |
| 7 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 4d | 4d | shed, batten, loom, config |
| 8 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 8d | 3d | burler, stencil |
| 9 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 4d | 4d | board, loom |
| 10 | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 4d | 4d | batten |
| 11 | **Git test fixtures trip Cortex XDR on the work laptop**<br>• Parallel git-heavy tests create so many files at once that Cortex XDR killed VS Code and then crashed itself, twice; tests must stay usable on a machine with endpoint protection | [`xdr-git-test-file-burst`](design-xdr-git-test-file-burst.md) | 3d | 13m | fabric |
| 12 | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 3d | 2d | stencil, shuttle |
| 13 | **Language-neutral gate: lyx gate run takes a slot for any command**<br>• The gate slot pool covers only go test; a C# or Python hub verify bypasses it. Add lyx gate run -- cmd and per-language parallelism. | [`gate-language-neutral`](design-gate-language-neutral.md) | 2d | 2d | config, webster, shed |
| 14 | **Entry gate on shed rows: a check decides whether a row runs at all**<br>• Replace darn recipe edge rewrite and the switched producer flag check with one shed entry gate per row; false means Done without acting. | [`row-entry-gate`](design-row-entry-gate.md) | 2d | 1d | shed, loom, batten |
| 15 | **A darn writer cannot rerun a verify tier it is refused, so a flaky tier failure loops and escalates** | [`darn-verify-tier-unrunnable`](design-darn-verify-tier-unrunnable.md) | 2d | 2d | loom, shed |
| 16 | **Remove the Webster kick-start pack**<br>• Drop every kick-start pack layout and its code; glyphs stay in the plan and its gates | [`kickstart-pack-remove`](design-kickstart-pack-remove.md) | 2d | 1d | webster, quarry, loom |
| 17 | **Use quarry Enclose for mechanical checks: verify failures to plan cards**<br>• Map tool-output file:line to member glyphs, then to the card and batch that changed them | [`enclose-checks`](design-enclose-checks.md) | 12d | 2d | quarry, webster, loom |
| 18 | **lyx board rename: change an entry slug in one write**<br>• Rename an entry keeping all fields and rewriting depends_on; report prose mentions | [`board-rename`](design-board-rename.md) | 2d | 2d | board |
| 19 | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 1d | 1d | webster, fabric, loom |
| 20 | **The orch watcher sees the note is written instead of waiting for a reply**<br>• At a context cycle the watcher checks that the orch turn ended and the note file exists, and cycles with no written reply; defer stays | [`orch-note-no-reply`](design-orch-note-no-reply.md) | 1d | 1d | orch, creel, workspaces |
| 21 | **Move the orch rules into the hub-wide rules file**<br>• Operator step after the orch-rules-files deploy: run lyx orch rules from the prime | [`orch-rules-move`](design-orch-rules-move.md) | 1d | 1d | orch |
| 22 | **Verify gates run only the touched trees and the repo-wide scans when a diff touches no production code**<br>• A diff that touches only a tool or docs still waits on both test tiers over the whole module at every verify gate | [`verify-diff-scope`](design-verify-diff-scope.md) | 1d | 1d | loom, shed, config |
| 23 | **Every remote operation runs async, and local-only fabric pairs stay an explicit choice only**<br>• Remote work goes async except the branch-exists check; batch pairs reach origin like any pair, and the local-only mode stays only as an explicit --local until a use is found or it is removed | [`fabric-remote-async`](design-fabric-remote-async.md) | 1d | 1d | fabric, batten |
| 24 | **lyx loom decision add notifies the run live LLM producer of the new decision**<br>• With the Go driver no agent lives for the whole run, so decision add tells the working LLM producer if any, and later producers and PR-Gate catch the rest | [`decision-add-notifies`](design-decision-add-notifies.md) | 1d | 1d | loom, creel |
| 25 | **Plan gate does not catch declared PATTERN.md growth that exceeds the headroom line** | [`plan-gate-pattern-headroom`](design-plan-gate-pattern-headroom.md) | 1d | 1d | loom, burler |
| 26 | **Plan cards mark additive edits so shared spots stop chaining the card DAG**<br>• An Extends label for edits that only add to a ref; two Extends on one ref are no dependency; measure with plandag first | [`plan-extends-label`](design-plan-extends-label.md) | 1d | 1d | loom, webster, burler |
| 27 | **lyx batten run starts in its own window by default**<br>• Every batten run is started with --window, so the window should be the default and foreground the opt-in. | [`batten-window-default`](design-batten-window-default.md) | 19h | 19h | batten, reed, orch |
| 28 | **A halt a plan card asks for reaches the parent as a question, not as a batch failure**<br>• A card-requested halt went through recover-batch, tackler and arbiter as a failure; it should go straight to the parent and the next attempt should carry the decision | [`card-halt-is-question`](design-card-halt-is-question.md) | 13h | 13h | webster, shed, loom |
| 29 | **A question research can settle is settled in Discussion, never deferred to Plan or an implementer**<br>• Discussion writer researches the facts its design rests on; reviewers flag a deferred research question or a halt-on-discovery rule | [`research-in-discussion`](design-research-in-discussion.md) | 13h | 13h | loom, stencil, burler |
| 30 | **prod pointer is shared across machines while each binary is local**<br>• A deploy on one machine moves origin/prod, leaving other machines silently behind it | [`prod-pointer-per-machine`](design-prod-pointer-per-machine.md) | 45m | 45m | deploy |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 31 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 2w | 9d | fabric |
| 32 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w |  | ide, fabric |
| 33 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 12d | 4d | shed |
| 34 | **[WS 007] An anchor per umbrella**<br>• Let an umbrella work in its own subdirectory of the repository, inherited by its tasks; hard to change now | [`umbrella-anchor`](design-umbrella-anchor.md) | 1d | 1d | fabric, config, workspaces |
| 35 | **reed warns on every strand launch on tmux 3.4 about allow-set-title**<br>• The optional allow-set-title fallback is right, but its Warn repeats per pane on a supported tmux | [`allow-set-title-warn-noise`](design-allow-set-title-warn-noise.md) | 42m | 42m | reed |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 006] Umbrella tasks: project threads that spawn and land from their own worktree**<br>• A larger project is an umbrella task with a long-lived pair and its own orch thread; tasks name it as parent, spawn from and land into its branch, and report to its thread; the prime is the root umbrella | [`orch-any-worktree`](design-orch-any-worktree.md) | 2w | 14h | fabric, batten, orch, board, reed, creel, shed, workspaces |
| 2 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 8d | 1d | webster |
| 3 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 4d | 1d | creel, shuttle, loom, stencil, report |
| 4 | **[WS 004] Board projects and pairs created from a parent pair**<br>• A task names its project; a project groups tasks in order and may have its own worktree; fabric and batten create a project task pair from that branch, batten verbs run from any worktree | [`pair-from-parent`](design-pair-from-parent.md) | 1d | 1d | fabric, batten, board, workspaces |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 2w | 3d | burler |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 2w |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 2w |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 2w | 2w | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 2w |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 2w |  | loom |
| 11 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w | 2w | fabric |
| 12 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 2w | 11d | config, board |
| 13 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 2w |  | shuttle |
| 14 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w | 2w | board, ide, orch |
| 15 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 11d | 7d | reed, shuttle, config |
| 16 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 4d | 4d | reed, config |
| 17 | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 3d | 2d | burler, shed, orch, quarry, report |
| 18 | **Regression tier: baseline profiles before the run, compare at the final gate**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 2d | 2d | shed, webster, batten, config |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 7d | 2d | shed, webster |
