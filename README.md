# Board

Rendered 2026-10-15 08:37 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Run status pane: numbered rows, seconds, round totals, gate time; segment-named sessions**<br>• Per-run status pane shows seconds, a total per Burler round, and each gate with its time and the rework it caused | [`run-pane-timing`](design-run-pane-timing.md) | loom | Discussion-Burler-Fix | 2h | 2h | loom, burler, shed, reed |
| 2 | **Begin-batch refusal route and label; bare lyx config writes an empty layer**<br>• Two small bugs: a delete-before-reference refusal is mislabelled plan_drifted and halts for a human; bare lyx config with no TTY saves an empty layer | [`refusal-route-config-empty`](design-refusal-route-config-empty.md) | loom | Discussion-Burler-Review | 3h | 26m | webster, loom, config |

### Ready

Waits on nothing open; can start now.

_None._

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Run window stays on shell: follow the overview and newest agent from run start**<br>• A viewer attached before any overview or agent window exists rests on shell and never follows the run | [`viewer-off-shell`](design-viewer-off-shell.md) | 37m | 33m | reed, batten |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **A hub pause leaves background forks and shells running, so hibernate never confirms**<br>• Pause interrupts turns but not background forks or shells; sessions read busy and hibernate confirmation fails | [`pause-background-work`](design-pause-background-work.md) | 12h | 2h | shuttle, webster |

### Enhancements

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Codex CLI as a second shuttle engine: hands-on spike, then codexengine**<br>• Codex CLI as a second engine to offload Claude tokens: short hands-on tests first, then a codexengine for single-session roles | [`codex-engine`](design-codex-engine.md) | 11d | 1d | shuttle |
| 2 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 10d | 10d | fabric, loom |
| 3 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 10d |  | shed |
| 4 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 9d | 9d | loom |
| 5 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 7d | 3d | quarry |
| 6 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | 4d | 4d | burler, stencil |
| 7 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) | 3d | 1h | loom, quarry, stencil |
| 8 | **lyx report timing: phase minutes, parent waits and agent spawns from run records**<br>• A mechanical per-run and per-day time report beside lyx report convergence, including stops by who settled them. | [`report-timing`](design-report-timing.md) | 2d | 2d | shed, orch |
| 9 | **Make shuttle and lyx neutral to the LLM provider**<br>• Engine registry, engine in run.json, no Claude names in the core, usage readers behind the engine | [`llm-agnostic-engine`](design-llm-agnostic-engine.md) | 2d | 1d | shuttle, config, stencil |
| 10 | **Tune the Burler lens fan prompts**<br>• Revisit the parked lens fan with lens and fork prompts that overlap less, measured against solo on the fan-lab fixtures | [`fan-prompt-tuning`](design-fan-prompt-tuning.md) | 1d | 1d | burler, stencil |
| 11 | **Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs** | [`dotnet-verify-protocol`](design-dotnet-verify-protocol.md) | 17h | 17h | loom, webster, config |
| 12 | **Run verbs in one shared namespace, not under lyx loom**<br>• lyx loom decision/status/goto/approve act on darn runs too; move recipe-neutral run verbs to one shared namespace | [`run-verbs-shared`](design-run-verbs-shared.md) | 15h | 15h | shed, loom |
| 13 | **Show the orch context size in the prime overview window**<br>• The prime overview window shows the orch session context tokens against its thresholds beside the batten runs | [`orch-context-overview`](design-orch-context-overview.md) | 14h | 14h | orch, batten |
| 14 | **Clearer loom model key names, one per step**<br>• Model keys like fix name the role they set, and every review segment gets its own reviewer and fixer key | [`loom-model-key-names`](design-loom-model-key-names.md) | 13h | 13h | config, loom |
| 15 | **Spawned code roles load the hub language skills**<br>• Darn, Webster, Burler and merge sessions write Go without scribe:golang-* skills; the hub config should name its language skills | [`role-language-skills`](design-role-language-skills.md) | 13h | 13h | config, shuttle |
| 16 | **Fork the Burler fixer from the reviewer session**<br>• The fixer re-reads the context the reviewer just read; fork it from the reviewer to reuse the cache | [`burler-fixer-fork`](design-burler-fixer-fork.md) | 2h | 2h | burler |
| 17 | **Webster fork read audit as a tool under tools/**<br>• Move the 2026-10-13 fork read audit jq scripts out of scratch into a repeatable tools/ script | [`glyph-audit-tool`](design-glyph-audit-tool.md) | 50m | 48m | webster |
| 18 | **The arbiter in shed for every shed run, batten included**<br>• The batten Go driver has a tackler but no arbiter; move the arbiter and its keys into shed beside the tackler | [`shed-arbiter-all`](design-shed-arbiter-all.md) | 5m | 3m | shed, batten, loom, config |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 11d | 4d | fabric |
| 20 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 10d |  | ide, fabric |
| 21 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 7d | 2d | shed |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 4d | 2d | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 2d | 1d | orch, batten, shed, loom |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | 12d | 7d | reed |
| 4 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 12d | 6d | burler |
| 5 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 11d |  | fabric, batten, orch |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 11d |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 11d |  | stencil |
| 8 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 11d | 9d | fabric |
| 9 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 11d |  | reed |
| 10 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 10d |  | loom |
| 11 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | 10d | 9d | webster, loom |
| 12 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 10d | 9d | fabric |
| 13 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 10d | 6d | config, board |
| 14 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 10d |  | shuttle |
| 15 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 10d | 9d | board, ide, orch |
| 16 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 6d | 2d | reed, shuttle, config |
| 17 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | 3d |  | orch |
| 18 | **Configurable action buttons on the reed status bar**<br>• Clickable status-bar buttons that run configured commands: VS Code, orch distill/refresh, the follow toggle and more | [`reed-status-buttons`](design-reed-status-buttons.md) | 33m | 28m | reed, config |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 2d | 2d | shed, webster |

### Audits

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit glyph use in Webster, the kick-start pack first**<br>• Check from transcripts of a smaller task whether Webster forks use the kick-start pack and glyphs as intended | [`glyph-use-audit`](design-glyph-use-audit.md) | 3d | 1d | webster, burler, quarry |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 4d | 1d |  |
| 3 | **Lighter test rules for small changes**<br>• Tests that pin stencil wording make small prompt changes large; pin only what must hold, measured first | [`lighter-test-rules`](design-lighter-test-rules.md) | 1d | 1d | stencil |
| 4 | **Bench fork0 and the strict critical fixer on twill Discussion-Write**<br>• Arms: twill as is, with the strict critical fixer, and fork0 plus the critical fixer, graded against Discussion-Review keys | [`twill-discussion-fork0-bench`](design-twill-discussion-fork0-bench.md) | 3h | 3h | loom, burler, stencil |
