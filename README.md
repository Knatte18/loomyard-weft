# Board

Rendered 2026-10-12 18:57 UTC; every age below is as of then.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### High priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Judge settles the review budget cap; arbiter only for stuck runs**<br>• At a spent review budget the judge tags findings mechanical or design-wrong and continues or accepts itself, instead of spawning the arbiter. | [`judge-cap-call`](design-judge-cap-call.md) | loom | Plan-Write | 4h | 55m | loom, burler |

#### Normal priority

| # | Task | Slug | Recipe | At | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | **reed: a strand in its own tmux window**<br>• Three related ideas for reed beyond one pane per strand and one window per worktree | [`reed-multi-window`](design-reed-multi-window.md) | loom | Webster-Burler-Review | 8d | 4h | reed, shuttle |
| 3 | **Go driver per task, with the tackler (cold assistant) and the arbiter behind it**<br>• Go steps the run and settles mechanical stops; a cheap assistant spawns only at stops no rule settles and may escalate to the arbiter; LLM driver stays as an option. | [`go-driver-assistant`](design-go-driver-assistant.md) | loom | Plan-Bouncer | 1h | 1h | loom, shed |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | **Run window and shuttle bugs from 2026-10-12**<br>• Six run window, shuttle, reed, loom and weft push bugs and fixes noted on 2026-10-12, bundled. | [`run-window-bugs`](design-run-window-bugs.md) | loom | `reed-multi-window` | 4d | 3h | shuttle, reed, loom, fabric |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Arbiter relaunch refused when the run already moved to the verdict row**<br>• A resume whose goto matches the row the run already stands on is refused as moved and parks the run. | [`arbiter-relaunch-moved`](design-arbiter-relaunch-moved.md) | 4h | 4h | loom, shed |
| 2 | **Plan does not surface tests that pin a method set a card extends** | [`plan-pinned-tests`](design-plan-pinned-tests.md) | 3h | 3h | loom |
| 3 | **Batten sends no notice for a landing that readies nothing**<br>• A plain landing is silent, so the orch misses landings the alignment agent approved. | [`batten-land-notice`](design-batten-land-notice.md) | 1h | 1h | batten |
| 4 | **Webster deviations list mixes in the batch's own files**<br>• The implementer stencil contradicts itself on deviations; let ScopeGuard alone produce the list | [`webster-deviations-list`](design-webster-deviations-list.md) | 12m | 12m | webster |

### Enhancements

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Audit glyph use in Webster, the kick-start pack first**<br>• Check from transcripts of a smaller task whether Webster forks use the kick-start pack and glyphs as intended | [`glyph-use-audit`](design-glyph-use-audit.md) | 11m | 0m | webster |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | 8d | 7d | fabric, loom |
| 3 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | 8d |  | shed |
| 4 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | 6d | 6d | loom |
| 5 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | 4d | 1d | quarry |
| 6 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | 2d | 2d | burler, stencil |
| 7 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` | 2d |  |  |
| 8 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) | 1d | 1d | loom, quarry, stencil |
| 9 | **Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default** | [`seats-replaces-singlellm`](design-seats-replaces-singlellm.md) | 1d | 5h | loom |
| 10 | **Measure the Webster kick-start pack with and without**<br>• Repeat the audit count (grep-for-member calls and whole-file reads per fork) on real loom runs with kickstart_pack on and off. | [`kickstart-pack-measure`](design-kickstart-pack-measure.md) | 1d | 1d | webster, burler, quarry |
| 11 | **Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss**<br>• After twill-plan lands: measure a lower plan_max_bounces, and cover the parent-directory miss every bench arm shared | [`twill-plan-followups`](design-twill-plan-followups.md) | 11h | 30m | loom |
| 12 | **Arbiter spawns only on stops that need it**<br>• Five arbiter spawns on 2026-10-12, none needed: narrow the triggers to the hard review cap, circling, and blocked or error stops with no computed way forward. | [`arbiter-when`](design-arbiter-when.md) | 4h | 3h | loom, shed |
| 13 | **Alignment agent settles landing order itself; escalates only on design conflict**<br>• Apply furthest-along-lands-first in the alignment agent and wake the orch only when two runs designs conflict. | [`alignment-landing-order`](design-alignment-landing-order.md) | 4h | 4h | orch, loom |
| 14 | **lyx report timing: phase minutes, parent waits and agent spawns from run records**<br>• A mechanical per-run and per-day time report beside lyx report convergence, including stops by who settled them. | [`report-timing`](design-report-timing.md) | 3h | 3h | shed, orch |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 15 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | 9d | 2d | fabric |
| 16 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | 8d |  | ide, fabric |
| 17 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | 5d | 5h | shed |
| 18 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | 2d | 11h | loom, stencil, config |

### Features

#### High priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | 1d | 5h | webster |
| 2 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | 8h | 49m | orch, batten, shed, loom |

#### Normal priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | 10d | 4d | reed |
| 4 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | 9d | 4d | burler |
| 5 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | 9d |  | fabric, batten, orch |
| 6 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | 9d |  | quarry |
| 7 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | 9d |  | stencil |
| 8 | **Codex as a second agent engine**<br>• If lyx gets a second engine, Codex is more relevant than Gemini | [`codex-engine`](design-codex-engine.md) | 9d | 7d | shuttle |
| 9 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | 9d | 7d | fabric |
| 10 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | 9d |  | reed |
| 11 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | 8d |  | loom |
| 12 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | 8d | 7d | webster, loom |
| 13 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | 8d | 7d | fabric |
| 14 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | 8d | 4d | config, board |
| 15 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | 8d |  | shuttle |
| 16 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | 8d | 7d | board, ide, orch |
| 17 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | 4d | 5h | reed, shuttle, config |
| 18 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | 1d |  | orch |

#### Low priority

| # | Note | Slug | Age | Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| 19 | **Scoped build and test for C# hubs on Windows**<br>• Verify gates in a .NET hub build and test only the projects a card touches, since a scanned Windows machine makes every build slow. | [`dotnet-scoped-verify`](design-dotnet-scoped-verify.md) | 7h | 7h | shed, webster |
