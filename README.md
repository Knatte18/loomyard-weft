# Board

Rendered 2026-10-15 12:11 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **lyx report timing: phase minutes, parent waits and agent spawns from run records**<br>• A mechanical per-run and per-day time report beside lyx report convergence, including stops by who settled them. | [`report-timing`](design-report-timing.md) | loom | Plan-Write | 2d | 44m | shed, orch |
| 2 | **Run window stays on shell: follow the overview and newest agent from run start**<br>• A viewer attached before any overview or agent window exists rests on shell and never follows the run | [`viewer-off-shell`](design-viewer-off-shell.md) | loom | Plan-Burler-Review | 4h | 43m | reed, batten |
| 3 | **Engine registry: engine per run, one constructor, narrower Engine interface**<br>• Shuttle picks the engine per spec from a registry and records it per run, so a second engine plugs in at one place | [`engine-registry`](design-engine-registry.md) | loom | Worktree-Create | 2d | 1h | shuttle, config, loom, webster, burler, orch, batten |
| 4 | **creel stage 1: the one way to reach an agent, standalone**<br>• The creel daemon, queues, journal and verbs, delivering through shuttle, built and tested before any module sends through it | [`creel-core`](design-creel-core.md) | loom | Plan-Write | 13d | 37m | creel, reed, shuttle |

### Ready

Waits on nothing open; can start now.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Webster master reads only what every fork shares, never source code**<br>• The master template tells the master to read the codebase once up front for its forks to inherit, but forks rewrite the code, so every later fork inherits stale reads | [`webster-master-no-code`](design-webster-master-no-code.md) | loom |  | 17m | 14m | webster, stencil |

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Plan cards name added tests by Create handle, never a plain test-file path**<br>• Drop the Edit exception for a test file the card only adds tests to; the Create plan handle carries the file | [`plan-test-file-path`](design-plan-test-file-path.md) | loom | `webster-master-no-code` | 3h | 3m | loom, stencil |
| 2 | **Engine-owned config, one neutral guardrail table, provider-blind reed**<br>• claude.yaml owned by claudeengine, a neutral shell-guard table each engine renders as hooks, reed strands tagged by provider | [`engine-config-seams`](design-engine-config-seams.md) | loom | `engine-registry` | 1h | 1h | shuttle, claude, config, reed |
| 3 | **Every role spec carries its resolved engine**<br>• Role specs take engine, model and effort from the model alias through one shared ModelChoice; Claude-only roles declare their needs | [`engine-model-choice`](design-engine-model-choice.md) | loom | `engine-registry` | 1h | 1h | shuttle, loom, webster, burler, shed |

### Layer B

Starts when every entry it names under After is done.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Lighter test rules for small changes**<br>• Tests that pin stencil wording make small prompt changes large; pin only what must hold, measured first | [`lighter-test-rules`](design-lighter-test-rules.md) | loom | `plan-test-file-path` | 1d | 3m | stencil, webster, burler, loom |
| 2 | **Usage and cost read through the engine**<br>• tokencount reads each run through its engine, with prices per engine, instead of reading Claude transcripts directly | [`engine-usage-readers`](design-engine-usage-readers.md) | loom | `engine-registry`, `engine-config-seams` | 1h | 1h | shuttle, claude |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Audits

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit every stencil for design assumptions the operator never meant**<br>• List each claim a stencil makes about how its role works, so the operator can confirm or reject it, after the Webster master code-read bug | [`stencil-intent-audit`](design-stencil-intent-audit.md) | 14m | 11m | stencil |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 4d | 1d |  |
| 3 | **Bench fork0 and the strict critical fixer on twill Discussion-Write**<br>• Arms: twill as is, with the strict critical fixer, and fork0 plus the critical fixer, graded against Discussion-Review keys | [`twill-discussion-fork0-bench`](design-twill-discussion-fork0-bench.md) | 6h | 6h | loom, burler, stencil |
| 4 | **Prototype reed status buttons in a throwaway tmux server**<br>• A fresh agent tries the menu button, follow toggle and extra button line by hand before reed-status-buttons becomes a task | [`reed-status-buttons-prototype`](design-reed-status-buttons-prototype.md) | 2h | 2h | reed |

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Webster fork deviations list mixes in its own target files**<br>• The implementer stencil asks a fork to compare its edits against the glyph union and also tells it not to, so deviations lists files that house its own target glyphs | [`webster-deviations-list`](design-webster-deviations-list.md) | 24m | 24m | webster, stencil |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **A hub pause leaves background forks and shells running, so hibernate never confirms**<br>• Pause interrupts turns but not background forks or shells; sessions read busy and hibernate confirmation fails | [`pause-background-work`](design-pause-background-work.md) | 16h | 6h | shuttle, webster |

### Enhancements

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 11d | 1h | shuttle, codex |
| 2 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 10d | 10d | fabric, loom |
| 3 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 10d |  | shed |
| 4 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 9d | 9d | loom |
| 5 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 7d | 3d | quarry |
| 6 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | 4d | 4d | burler, stencil |
| 7 | **Tune the Burler lens fan prompts**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | 2d | 2d | burler, stencil |
| 8 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 21h | 21h | loom, webster, config |
| 9 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 18h | 18h | shed, loom |
| 10 | **Clearer loom model key names, one per step**<br>• Model keys like fix name the role they set, and every review segment gets its own reviewer and fixer key | [`loom-model-key-names`](design-loom-model-key-names.md) | 17h | 17h | config, loom |
| 11 | **Fork the Burler fixer from the reviewer session**<br>• The fixer re-reads the context the reviewer just read; fork it from the reviewer to reuse the cache | [`burler-fixer-fork`](design-burler-fixer-fork.md) | 6h | 6h | burler |
| 12 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 3h | 3h | shed, batten, loom, config |
| 13 | **Paste cited code into a review rounds context**<br>• Reviewers edit nothing and review a fixed commit, so code pasted at round start cannot go stale; measure it against todays reviewer | [`review-inline-code`](design-review-inline-code.md) | 3d | 24m | burler, stencil |
| 14 | **Module labels from what a task touches, not only what it is about**<br>• Half the landings that changed the shuttle core lacked the shuttle label; derive or check module labels from the plan and the diff | [`labels-from-diff`](design-labels-from-diff.md) | 1h | 1h | board, loom |
| 15 | **Check overlap between tasks that wait on the same entry**<br>• Four tasks waited on kickstart-per-card and would have auto-started together although three edit the same stencils; nothing checks overlap among siblings | [`sibling-overlap-check`](design-sibling-overlap-check.md) | 2m | 0m | board, batten, orch |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 16 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 11d | 5d | fabric |
| 17 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 10d |  | ide, fabric |
| 18 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 7d | 2d | shed |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4d | 2d | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 3d | 2d | orch, batten, shed, loom |
| 3 | **creel stage 2: orch and batten notices through creel**<br>• The orch watcher and batten notices and injects go through creel; the watcher keeps only its cycle logic | [`creel-notices`](design-creel-notices.md) | 1h | 34m | creel, orch, batten |
| 4 | **creel stage 3: agents message through creel, not SendMessage**<br>• Parent asks, parent review, stop reports and shuttle send go through creel, so a non-Claude engine can take a role with a parent | [`creel-agents`](design-creel-agents.md) | 1h | 34m | creel, shuttle, loom, stencil |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 5 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 12d | 6d | burler |
| 6 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 12d |  | fabric, batten, orch |
| 7 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 11d |  | quarry |
| 8 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 11d |  | stencil |
| 9 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 11d | 9d | fabric |
| 10 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 11d |  | reed |
| 11 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 10d |  | loom |
| 12 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | 10d | 9d | webster, loom |
| 13 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 10d | 9d | fabric |
| 14 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 10d | 6d | config, board |
| 15 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 10d |  | shuttle |
| 16 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 10d | 9d | board, ide, orch |
| 17 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 6d | 2d | reed, shuttle, config |
| 18 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | 3d |  | orch |
| 19 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 4h | 4h | reed, config |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 20 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 3d | 3d | shed, webster |
