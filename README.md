# Board

Rendered 2026-10-18 16:28 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Fast batten start, local-only fabric pairs and one Command producer**<br>• Batten pushes status to GitHub after every row (about 2 s each); fabric add/remove wait on origin even for local pairs; side-action rows become one Command producer | [`batten-start-speed`](design-batten-start-speed.md) | loom | Webster | 1d | 9h | batten, fabric, shed, config, loom |
| 2 | **Named run profiles in one hub file, picked per task by a profile field**<br>• A profiles.yaml on Bolt maps names to run_settings-shaped sets; a task names one in its profile field, and batten seeds it under the task run_settings<br>• `darn: review_max_bounces 1` | [`run-profiles`](design-run-profiles.md) | darn | Merge-In | 1h | 30m | board, batten, config, loom |
| 3 | **tokencount reports cost per segment, most expensive first, each split into its parts**<br>• Lead the tokencount report with the most expensive parts and a per-run segment table, and show roles as segment totals with sub-rows<br>• `darn: review_max_bounces 0` | [`tokencount-segment-report`](design-tokencount-segment-report.md) | darn | Darn-Write | 0m | 0m | report |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### High priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **[WS 002] One VS Code workspace per umbrella, listing its worktrees directly**<br>• Workspace files in the hub dir; umbrella worktree first, then Bolt, then its live pairs at their anchors; no _portals; finished pairs drop out<br>• `loom: discussion_max_bounces 2`<br>• `loom: merge_max_bounces 1`<br>• `loom: plan_max_bounces 1`<br>• `loom: webster_max_bounces 1` | [`hub-workspaces`](design-hub-workspaces.md) | loom | `batten-start-speed` | 2h | 1h | fabric, ide, workspaces |

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | **Implementer seam: batchifier inside Webster, engine chosen by setting**<br>• Groundwork before Weaver: batching moves into the implementer, a written implementer contract, loom.implementer selects the engine<br>• `darn: review_max_bounces 1` | [`implementer-seam`](design-implementer-seam.md) | darn | `batten-start-speed` | 1d | 9h | webster, loom, shed, report |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 3d | 3d | reed |
| 2 | **Classify Plan lens and review findings to make the Plan writer get it right first**<br>• Find what Plan lenses and Plan-Burler-Review keep catching, and stop each recurring class at the writer | [`plan-writer-finding-classes`](design-plan-writer-finding-classes.md) | 2d | 2d | loom, burler, stencil, shed |
| 3 | **Recurring test-suite time and growth audit**<br>• Re-run cmd/testtiming against test-suite-timing.md to watch suite growth | [`test-time-audit-recurring`](design-test-time-audit-recurring.md) | 2h | 2h |  |
| 4 | **Audit the use of the twill and lens review setup at every perch**<br>• After about ten loom runs, measure what the lens fans and twill fixers catch per perch against their cost, and set the standard review rounds from it | [`review-setup-audit`](design-review-setup-audit.md) | 1h | 1h | burler, loom, report |

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **webster create-not-done check rejects a Create target the card removes itself** | [`create-target-transient`](design-create-target-transient.md) | 30m | 30m | webster, loom, quarry |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 005] One dobby daemon per beam owns and drives its runs**<br>• Each beam (a worktree with an orch thread) gets a Go daemon, started by lyx orch start, that follows its runs status files, starts ready tasks, lands, settles mechanical stops and tells the thread only what needs judgement | [`umbrella-daemon`](design-umbrella-daemon.md) | 6d | 36m | batten, orch, shed, workspaces, loom, board |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 2w | 3d | shuttle, codex |
| 3 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 2w | 13d | fabric, loom |
| 4 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 2w | 2d | loom, webster, config |
| 5 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 3d | 3d | shed, loom |
| 6 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 3d | 3d | shed, batten, loom, config |
| 7 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 7d | 2d | burler, stencil |
| 8 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 3d | 3d | board, loom |
| 9 | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 3d | 3d | batten |
| 10 | **Git test fixtures trip Cortex XDR on the work laptop**<br>• Parallel git-heavy tests create so many files at once that Cortex XDR killed VS Code and then crashed itself, twice; tests must stay usable on a machine with endpoint protection | [`xdr-git-test-file-burst`](design-xdr-git-test-file-burst.md) | 2d | 2d | fabric |
| 11 | **Rename the lyx binary to lyr (name clash with LyX)** | [`rename-binary-lyr`](design-rename-binary-lyr.md) | 2d | 1d | stencil, shuttle |
| 12 | **Language-neutral gate: lyx gate run takes a slot for any command**<br>• The gate slot pool covers only go test; a C# or Python hub verify bypasses it. Add lyx gate run -- cmd and per-language parallelism. | [`gate-language-neutral`](design-gate-language-neutral.md) | 1d | 1d | config, webster, shed |
| 13 | **Entry gate on shed rows: a check decides whether a row runs at all**<br>• Replace darn recipe edge rewrite and the switched producer flag check with one shed entry gate per row; false means Done without acting. | [`row-entry-gate`](design-row-entry-gate.md) | 1d | 9h | shed, loom, batten |
| 14 | **Preflight pushes the weft branch synchronously and races the board sync**<br>• Preflight spent 11 s mostly in a weft push to GitHub, refused once after 8 s; push detached or serialise with the board sync. | [`preflight-weft-push-sync`](design-preflight-weft-push-sync.md) | 1d | 1d | fabric, shed |
| 15 | **A darn writer cannot rerun a verify tier it is refused, so a flaky tier failure loops and escalates** | [`darn-verify-tier-unrunnable`](design-darn-verify-tier-unrunnable.md) | 1d | 1d | loom, shed |
| 16 | **Remove the Webster kick-start pack**<br>• Drop every kick-start pack layout and its code; glyphs stay in the plan and its gates | [`kickstart-pack-remove`](design-kickstart-pack-remove.md) | 1d | 1h | webster, quarry, loom |
| 17 | **Use quarry Enclose for mechanical checks: verify failures to plan cards**<br>• Map tool-output file:line to member glyphs, then to the card and batch that changed them | [`enclose-checks`](design-enclose-checks.md) | 10d | 1d | quarry, webster, loom |
| 18 | **lyx board rename: change an entry slug in one write**<br>• Rename an entry keeping all fields and rewriting depends_on; report prose mentions | [`board-rename`](design-board-rename.md) | 1d | 1d | board |
| 19 | **Parallel batches: the shared pieces for Webster forks and Weaver**<br>• DAG batchifier, worktree per batch, merge-back with conflict resolution; open: keeping a fork inside its worktree | [`parallel-batch-parts`](design-parallel-batch-parts.md) | 9h | 9h | webster, fabric, loom |
| 20 | **The orch watcher sees the note is written instead of waiting for a reply**<br>• At a context cycle the watcher checks that the orch turn ended and the note file exists, and cycles with no written reply; defer stays | [`orch-note-no-reply`](design-orch-note-no-reply.md) | 1h | 1h | orch, creel, workspaces |
| 21 | **Move the orch rules into the hub-wide rules file**<br>• Operator step after the orch-rules-files deploy: run lyx orch rules from the prime | [`orch-rules-move`](design-orch-rules-move.md) | 48m | 48m | orch |
| 22 | **A batten verb that ends a paused or blocked run and removes its records**<br>• Dead paused and blocked run records stay in the records and the overview, since no verb ends them and cleanup skips them | [`batten-abandon`](design-batten-abandon.md) | 16m | 16m | batten, fabric |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 23 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 2w | 8d | fabric |
| 24 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 2w |  | ide, fabric |
| 25 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 11d | 2d | shed |
| 26 | **A hub pause leaves background forks and shells running, so hibernate never confirms**<br>• Pause interrupts turns but not background forks or shells; sessions read busy and hibernate confirmation fails | [`pause-background-work`](design-pause-background-work.md) | 3d | 1d | shuttle, webster |
| 27 | **[WS 007] An anchor per umbrella**<br>• Let an umbrella work in its own subdirectory of the repository, inherited by its tasks; hard to change now | [`umbrella-anchor`](design-umbrella-anchor.md) | 2h | 2h | fabric, config, workspaces |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **[WS 006] Umbrella tasks: project threads that spawn and land from their own worktree**<br>• A larger project is an umbrella task with a long-lived pair and its own orch thread; tasks name it as parent, spawn from and land into its branch, and report to its thread; the prime is the root umbrella | [`orch-any-worktree`](design-orch-any-worktree.md) | 2w | 56m | fabric, batten, orch, board, reed, creel, shed, workspaces |
| 2 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 7d | 8h | webster |
| 3 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 3d | 1d | creel, shuttle, loom, stencil, report |
| 4 | **[WS 004] Board projects and pairs created from a parent pair**<br>• A task names its project; a project groups tasks in order and may have its own worktree; fabric and batten create a project task pair from that branch, batten verbs run from any worktree | [`pair-from-parent`](design-pair-from-parent.md) | 2h | 1h | fabric, batten, board, workspaces |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 2w | 2d | burler |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 2w |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 2w |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 2w | 12d | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 2w |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 2w |  | loom |
| 11 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 2w | 12d | fabric |
| 12 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 2w | 9d | config, board |
| 13 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 2w |  | shuttle |
| 14 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 2w | 12d | board, ide, orch |
| 15 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 9d | 6d | reed, shuttle, config |
| 16 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 3d | 3d | reed, config |
| 17 | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 2d | 1d | burler, shed, orch, quarry, report |
| 18 | **Regression tier: baseline profiles before the run, compare at the final gate**<br>• Support a hub regression tier that writes profiles on the parent before implementation and compares after, switchable per task for expected drift. | [`regression-baseline-verify`](design-regression-baseline-verify.md) | 1d | 1d | shed, webster, batten, config |
| 19 | **Draw a plan as a card DAG with its glyph and file matrix**<br>• A tools/ program that reads a plan, builds the card x glyph/file matrix and writes Graphviz DOT of the card dependencies | [`plan-dag-view`](design-plan-dag-view.md) | 1d | 1d | loom, webster |
| 20 | **Go-driven loom run: no long-lived LLM driver per run, and who receives a run message**<br>• A Go loop steps every loom run and spawns a cold LLM watcher only on a stop it cannot pass; messages for a run go to its driver or a durable record | [`go-driven-loom`](design-go-driven-loom.md) | 1h | 1h | loom, shed, creel |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 21 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 6d | 1d | shed, webster |
