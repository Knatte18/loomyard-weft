# Board

Rendered 2026-10-16 14:33 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### High priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Burler lens fan and twill fixer in production, per-segment lenses for Discussion and Plan review**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | loom | Webster-Bouncer | 6d | 2h | burler, stencil, loom |

### Ready

Waits on nothing open; can start now.

_None._

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 6d | 3d |  |
| 2 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 1d | 1d | reed |
| 3 | **Classify Plan lens and review findings to make the Plan writer get it right first**<br>• Find what Plan lenses and Plan-Burler-Review keep catching, and stop each recurring class at the writer | [`plan-writer-finding-classes`](design-plan-writer-finding-classes.md) | 20h | 2h | loom, burler, stencil, shed |

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Reusing a landed task's slug: Publish finds the old run's closed PR and halts** | [`slug-reuse-old-pr`](design-slug-reuse-old-pr.md) | 3h | 3h | loom, batten, board |
| 2 | **A note that only names a running task stops the alignment agent from settling its PR**<br>• Alignment reads any note naming the slug as a recorded decision | [`alignment-note-mention-trigger`](design-alignment-note-mention-trigger.md) | 3h | 3h | loom |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **A hub pause leaves background forks and shells running, so hibernate never confirms**<br>• Pause interrupts turns but not background forks or shells; sessions read busy and hibernate confirmation fails | [`pause-background-work`](design-pause-background-work.md) | 1d | 1d | shuttle, webster |

### Enhancements

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 12d | 1d | shuttle, codex |
| 2 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 12d | 11d | fabric, loom |
| 3 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 12d |  | shed |
| 4 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 10d | 10d | loom |
| 5 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 8d | 1d | quarry |
| 6 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 12d | 19h | loom, webster, config |
| 7 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 1d | 1d | shed, loom |
| 8 | **Clearer loom model key names, one per step**<br>• Model keys like fix name the role they set, and every review segment gets its own reviewer and fixer key | [`loom-model-key-names`](design-loom-model-key-names.md) | 1d | 1d | config, loom |
| 9 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 1d | 1d | shed, batten, loom, config |
| 10 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 4d | 6h | burler, stencil |
| 11 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 1d | 1d | board, loom |
| 12 | **Start one task per overlapping group at a newly-ready start**<br>• When a landing makes several overlapping tasks ready at once, batten starts one per group and names the rest in its notice | [`batten-ready-overlap`](design-batten-ready-overlap.md) | 1d | 1d | batten |
| 13 | **Rename the `lyx` binary to `lyd` (name clash with LyX)** | [`rename-binary-lyd`](design-rename-binary-lyd.md) | 7h | 7h | stencil, shuttle |
| 14 | **Rewrite C13's stacked half for window-per-strand**<br>• C13's stacked 3-row half cannot be set up once reed-add-parentless-placement lands | [`creel-c13-stacked-rewrite`](design-creel-c13-stacked-rewrite.md) | 5h | 5h | creel |
| 15 | **Go driver messages the orch about a verify halt batten retries itself**<br>• Drop or reword the parent message for a watched verify halt | [`driver-watched-verify-notice`](design-driver-watched-verify-notice.md) | 4h | 4h | shed, batten |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 16 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 12d | 6d | fabric |
| 17 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 12d |  | ide, fabric |
| 18 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 8d | 19h | shed |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 5d | 4d | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 4d | 3d | orch, batten, shed, loom |
| 3 | **creel stage 2: orch and batten notices through creel**<br>• The orch watcher and batten notices and injects go through creel; the watcher keeps only its cycle logic | [`creel-notices`](design-creel-notices.md) | 1d | 1d | creel, orch, batten |
| 4 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 1d | 1d | creel, shuttle, loom, stencil |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 13d | 6h | burler |
| 6 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 13d |  | fabric, batten, orch |
| 7 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 12d |  | quarry |
| 8 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 12d |  | stencil |
| 9 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 12d | 10d | fabric |
| 10 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 12d |  | reed |
| 11 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 12d |  | loom |
| 12 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 12d | 10d | fabric |
| 13 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 12d | 7d | config, board |
| 14 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 12d |  | shuttle |
| 15 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 11d | 10d | board, ide, orch |
| 16 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 7d | 4d | reed, shuttle, config |
| 17 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 1d | 1d | reed, config |
| 18 | **Ideas to borrow from comparable agent orchestrators (survey, Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 7h | 7h | burler, shed, orch, quarry |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 4d | 4d | shed, webster |
