# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### High priority

| # | Task | Slug | Recipe | At | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Merge review perch and a parent merge lock that survives sleep and crashes**<br>• When the parent moved while a task ran, loom has a cold-start agent review the result of merging the parent in, since no earlier review saw that interaction. | [`merge-review`](design-merge-review.md) | loom | Plan-Write | loom, burler, batten, shed |
| 2 | **Config layers and per-task run settings**<br>• Config resolves through a repo default, the worktree file and an untracked local override under .lyx/; a board task carries run settings that batten writes into the pair local override. | [`config-layers-run-settings`](design-config-layers-run-settings.md) | loom | Plan-Burler-Fix | config, batten, board |
| 3 | **Twill for Plan-Write at opus[medium]**<br>• Bench a self-forking lens review for the plan writer, as twill does for Discussion | [`twill-plan`](design-twill-plan.md) | loom | Plan-Write | loom, stencil |
| 4 | **Agent shell hygiene: read-only sed allowed, no scripted edits or /tmp rm in Webster forks**<br>• Make read-only sed work without refusals or prompts across lyx-spawned agents, stencils and audits, keeping the no-script-edit rule. | [`sed-read-ok`](design-sed-read-ok.md) | loom | Discussion-Write | stencil, webster, shuttle |
| 5 | **lyx cleans up after itself: temp files and one cleanup command**<br>• Tests and fixtures leave GBs of temps in /tmp; remove them on exit, sweep leftovers, and add lyx cleanup running every module cleanup | [`test-tmp-cleanup`](design-test-tmp-cleanup.md) | loom | Plan-Write | fabric, loom, batten, board |

#### Normal priority

| # | Task | Slug | Recipe | At | Labels |
| --- | --- | --- | --- | --- | --- |
| 6 | **Board entries carry created and revised dates; views show age**<br>• Entries carry created and updated dates; list and README show age and can sort by it, so long-lying entries stand out | [`board-entry-age`](design-board-entry-age.md) | darn | Darn-Burler-Review | board |

### Ready

Waits on nothing open; can start now.

_None._

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Plan gate delete-before-reference matches deleted methods by bare name**<br>• Hundreds of false blocking findings when a plan deletes a method with a common name | [`delete-before-reference-receiver`](design-delete-before-reference-receiver.md) | quarry, loom |
| 2 | **Plan-review fixes are proposed without running lyx loom validate-plan; fix glyphs name the wrong function** | [`plan-review-validate`](design-plan-review-validate.md) | loom, stencil |

### Enhancements

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Alignment agent in loom checks a run against the board and runs in flight**<br>• A cold-start agent at parent review and PR-Gate reads the board and in-flight runs records and checks alignment, waking the orch only on an overlap. | [`alignment-agent`](design-alignment-agent.md) | orch, loom |

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 2 | **Convergence: log hub interventions and driver repairs per run by class**<br>• Record every hub intervention and driver repair with a class and cause, so interventions per run and repeat failure classes are visible | [`convergence-plan`](design-convergence-plan.md) | orch, shed |
| 3 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | fabric, loom |
| 4 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | shed |
| 5 | **lyx seeds the prime VS Code setup: title, orch start task, keybindings**<br>• LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json | [`ide-prime-setup`](design-ide-prime-setup.md) | ide, orch |
| 6 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | loom |
| 7 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | shed |
| 8 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | quarry |
| 9 | **Shuttle: the launch line carries the first turn**<br>• /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed | [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) | shuttle |
| 10 | **reed: strands state their size; parallel agents share the window evenly**<br>• Replace the one collapse-all-but-the-newest rule with per-strand sizing: content-sized panes like the runs overview, and parallel agents that share space evenly | [`reed-layout-shares`](design-reed-layout-shares.md) | reed |
| 11 | **Config-driven file log level**<br>• logger.yaml gains a level key for the file trace, independent of -v, so a crash can be traced without a rebuild | [`logger-file-level`](design-logger-file-level.md) | config |
| 12 | **One owner for child-process lifetime in internal/proc**<br>• Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill | [`proc-child-lifetime`](design-proc-child-lifetime.md) |  |
| 13 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | burler, stencil |
| 14 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` |  |
| 15 | **Discussion chair asks advisors open questions without its options or pick** | [`chair-blind-questions`](design-chair-blind-questions.md) | loom, stencil |
| 16 | **A paste over tmux message size fails whole, with no chunking** | [`paste-size-limit`](design-paste-size-limit.md) | shuttle, reed |
| 17 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) | loom, quarry, stencil |
| 18 | **Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default** | [`seats-replaces-singlellm`](design-seats-replaces-singlellm.md) | loom |
| 19 | **tokencount: guard the slug-reuse cutoff against date skew**<br>• The slug-reuse cutoff is always applied; a tag commit date skewed against transcript timestamps could silently drop sessions. | [`tokencount-cutoff-skew`](design-tokencount-cutoff-skew.md) |  |
| 20 | **Sweep tests that rely on a closed listener socket file**<br>• A forked child of a parallel test can inherit a closed listener fd and keep a stale socket answering; sweep other tests for the same race. | [`reed-socket-test-fd-sweep`](design-reed-socket-test-fd-sweep.md) | reed |
| 21 | **End the seat hold deferral when the advisor itself is held or dead**<br>• The hold notice deferral only got a shorter cap (10 min); ending it as soon as the advisor is held or dead is not built. | [`hold-deferral-end-on-held`](design-hold-deferral-end-on-held.md) | shuttle |
| 22 | **Measure the Webster kick-start pack with and without**<br>• Repeat the audit count (grep-for-member calls and whole-file reads per fork) on real loom runs with kickstart_pack on and off. | [`kickstart-pack-measure`](design-kickstart-pack-measure.md) | webster |
| 23 | **Twill for Plan: measure fewer Plan-Review rounds; wrong-parent-dir miss**<br>• After twill-plan lands: measure a lower plan_max_bounces, and cover the parent-directory miss every bench arm shared | [`twill-plan-followups`](design-twill-plan-followups.md) | loom |

#### Low priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 24 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | fabric |
| 25 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | ide, fabric |
| 26 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | loom, stencil |
| 27 | **Discussion advisors: operator-named areas per advisor on top of the landed all-advisors default**<br>• Each discussion_advisors entry takes an optional area (a lens, as in review fans); without one the advisor answers every question | [`discussion-advisor-mode`](design-discussion-advisor-mode.md) | loom, stencil, config |
| 28 | **Fabric audit: plain readers accept unquoted globs and any flag**<br>• readOnlyCommands readers (cat, grep -r) accept unquoted globs and arbitrary flags | [`audit-reader-globs`](design-audit-reader-globs.md) | webster, fabric |

### Features

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **reed: runs overview pane above the orch; Selvage reachable by key**<br>• A minimal pane under the orch listing every run in flight and its shed state; the per-run batten windows stay but the status bar hides them | [`runs-overview`](design-runs-overview.md) | reed, orch, batten |
| 2 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | webster |
| 3 | **Split the orch into a run driver and a conversational orch**<br>• One agent drives runs and takes escalations; the operator talks to the other, which discusses problems and updates the board. | [`orch-split`](design-orch-split.md) | orch, batten |

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 4 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | reed |
| 5 | **Kick-start pack for Burler: pre-resolved glyph spans before the explore phase**<br>• Burler side of the kick-start pack (#228); the Webster side is in task plan-glyph-tuning | [`kick-start-pack`](design-kick-start-pack.md) | burler, quarry |
| 6 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | burler |
| 7 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | fabric, batten, orch |
| 8 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | quarry |
| 9 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | stencil |
| 10 | **Codex as a second agent engine**<br>• If lyx gets a second engine, Codex is more relevant than Gemini | [`codex-engine`](design-codex-engine.md) | shuttle |
| 11 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | fabric |
| 12 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | reed |
| 13 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | loom |
| 14 | **webster: worktree-per-card parallel execution**<br>• Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays | [`webster-parallel-execution`](design-webster-parallel-execution.md) | webster |
| 15 | **reed: cross-worktree columns, own-window anchoring, per-window attach**<br>• Three related ideas for reed beyond one pane per strand and one window per worktree | [`reed-multi-window`](design-reed-multi-window.md) | reed |
| 16 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | webster, loom |
| 17 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | fabric |
| 18 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | config, board |
| 19 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | shuttle |
| 20 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | board, ide |
| 21 | **Shortcut commands built into lyx from one committed file**<br>• An embedded shortcut file, git-alias style: attach for reed attach, and one shortcut that starts the orch and attaches in the prime | [`operator-shortcuts`](design-operator-shortcuts.md) | orch, reed |
| 22 | **Open an extra named agent strand through shuttle**<br>• Open a named Claude session in a strand with no task, through shuttle, so it keeps its name; for extra strands in the prime | [`extra-agent-strand`](design-extra-agent-strand.md) | shuttle |
| 23 | **Who in a running task receives an instruction**<br>• Driver forwarding, an alias for the active strand kept by loom, or an on-call LLM per run: open for a brainstorm | [`run-message-routing`](design-run-message-routing.md) | loom |
| 24 | **A pane listing every gate of a run as a checklist with timings**<br>• Every gate of the run with a tick box and its elapsed time, in a pane of its own; an idea, not a priority | [`gate-checklist-pane`](design-gate-checklist-pane.md) | reed |
| 25 | **lyx pause: stop every run at once, safely and on purpose**<br>• A hub-wide pause that interrupts every Claude session with Esc within seconds, unlike the step-boundary pauses, recorded as ordered so no failure or idle path fires | [`lyx-halt`](design-lyx-halt.md) | batten, shuttle |
| 26 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | reed, shuttle |
| 27 | **lyx orch start --continue resumes the last orch session**<br>• lyx orch records its session id so a reboot resumes the same orch conversation instead of a fresh one. | [`orch-continue`](design-orch-continue.md) | orch |

#### Low priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 28 | **Plan-Write on MultiLLMProducer: advisors gather facts and check cards**<br>• Later: Plan-Write as the chair, with an advisor that maps callers, tests and verify commands per package, and possibly one that checks each card as it is written | [`plan-multi-llm`](design-plan-multi-llm.md) | loom, stencil |
