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
| 1 | **Webster forks wake Merriam on every gate wait**<br>• Forks end their turn while a background gate runs, so Merriam gets a finished notice and a wasted turn each time; quick checks and long paths add more | [`fork-wait-noise`](design-fork-wait-noise.md) | loom | Discussion-Burler | webster, stencil |

#### Normal priority

| # | Task | Slug | Recipe | At | Labels |
| --- | --- | --- | --- | --- | --- |
| 2 | **Shuttle start and seat holds: tolerate a stray % at the prompt, probe every second, cap the seat hold deferral**<br>• A ready pane with a stray % fails the producer, awaitStartup sleeps 10 s between checks, and a seat chair hold deferral of 20 min can hide a wedged advisor | [`shuttle-startup-ready`](design-shuttle-startup-ready.md) | loom | Discussion-Burler | shuttle, batten |
| 3 | **Small fixes: tokencount gaps and slug reuse, loom step help, plan rubric file-target rule, batten notice state**<br>• Five small independent fixes across tools/tokencount, loomcli help, the plan review rubric and batten notices | [`small-fixes-oct11`](design-small-fixes-oct11.md) | loom | Discussion-Write | shed, loom, burler, stencil, batten |

### Ready

Waits on nothing open; can start now.

_None._

### Layer A

Waits only on Running or Ready entries.

#### Normal priority

| # | Task | Slug | Recipe | After | Labels |
| --- | --- | --- | --- | --- | --- |
| 1 | **Parent-review wait: warm writer cache on reject, wait bound off the boundary, gate hardening**<br>• A late reject resumes the writer on a cold cache, a writer stuck mid-turn outlives the wait bound, and the PR #336 review left gate gaps | [`parent-review-wait`](design-parent-review-wait.md) | loom | `shuttle-startup-ready` | loom, shuttle, orch |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-twc-1`](design-twb-twc-1.md) | webster, shuttle, stencil, quarry |
| 2 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-twc-2`](design-twb-twc-2.md) | webster, shuttle, stencil, quarry |
| 3 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-combo-1`](design-twb-combo-1.md) | webster, shuttle, stencil, quarry |
| 4 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-combo-2`](design-twb-combo-2.md) | webster, shuttle, stencil, quarry |
| 5 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-twp-1`](design-twb-twp-1.md) | webster, shuttle, stencil, quarry |
| 6 | **Webster stability: gate shells, Master self-fix, recovery, Create checks**<br>• Fix the Webster faults outside the audit that stop or mislead runs | [`twb-twp-2`](design-twb-twp-2.md) | webster, shuttle, stencil, quarry |

### Enhancements

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Convergence: log hub interventions and driver repairs per run by class**<br>• Record every hub intervention and driver repair with a class and cause, so interventions per run and repeat failure classes are visible | [`convergence-plan`](design-convergence-plan.md) | orch, shed |
| 2 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | fabric, loom |
| 3 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | shed |
| 4 | **lyx seeds the prime VS Code setup: title, orch start task, keybindings**<br>• LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json | [`ide-prime-setup`](design-ide-prime-setup.md) | ide, orch |
| 5 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | loom |
| 6 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | shed |
| 7 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | quarry |
| 8 | **Audit how plans use glyphs in practice, then tune the plan stencil**<br>• First audit done (findings below); re-run it by one agent after a good number of runs land on the implementer quarry hint (a28e79d9c) | [`glyph-practice-audit`](design-glyph-practice-audit.md) | loom, stencil |
| 9 | **Capture real hook payloads into fixtures**<br>• Run the llm-tier hook-capture scenario once and turn the real StopFailure, Notification and SessionEnd payloads into corpus fixtures | [`shuttle-hook-payload-corpus`](design-shuttle-hook-payload-corpus.md) | shuttle |
| 10 | **Shuttle: the launch line carries the first turn**<br>• /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed | [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) | shuttle |
| 11 | **reed: strands state their size; parallel agents share the window evenly**<br>• Replace the one collapse-all-but-the-newest rule with per-strand sizing: content-sized panes like the runs overview, and parallel agents that share space evenly | [`reed-layout-shares`](design-reed-layout-shares.md) | reed |
| 12 | **Config-driven file log level**<br>• logger.yaml gains a level key for the file trace, independent of -v, so a crash can be traced without a rebuild | [`logger-file-level`](design-logger-file-level.md) | config |
| 13 | **One owner for child-process lifetime in internal/proc**<br>• Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill | [`proc-child-lifetime`](design-proc-child-lifetime.md) |  |
| 14 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) | burler, stencil |
| 15 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` |  |
| 16 | **Reviewer treats the kick-start pack as its read set, with a budget beyond it**<br>• Fan-lab 2026-10-10: a solo opus[medium] reviewer given a kick-start pack ("read all listed spans first") read the pack and then explored about 20 more tool calls, so the pack came on top of exploration instead of replacing it; neither the prompt nor the ladder card said to stay with the pack. Idea: the prompt says the pack is the read set and is trusted as read; any read beyond it needs a named gap (a caller or spawned process one hop out), within a small tool-call budget, and the reviewer ends with one line per extra read saying why the pack lacked it (the fork rule from fan-master-preread variant D). | `pack-is-the-read-set` | burler, stencil |
| 17 | **Cut the orch's token spend**<br>• Wake the orch less: an alignment agent in loom reviews against the board, waking the orch only on overlap | [`orch-token-spend`](design-orch-token-spend.md) | orch, loom |
| 18 | **Discussion chair asks advisors open questions without its options or pick** | [`chair-blind-questions`](design-chair-blind-questions.md) | loom, stencil |
| 19 | **A paste over tmux message size fails whole, with no chunking** | [`paste-size-limit`](design-paste-size-limit.md) | shuttle, reed |
| 20 | **No live test that a paste followed by Enter submits in Claude Code** | [`paste-submit-live-check`](design-paste-submit-live-check.md) | shuttle |
| 21 | **One seats Discussion producer: advisor count and lens forks as config axes** | [`seats-twill-combined`](design-seats-twill-combined.md) | loom, stencil, config |
| 22 | **Discussion premises cite the glyph they rest on; the gate resolves them** | [`discussion-premise-cites`](design-discussion-premise-cites.md) | loom, quarry, stencil |

#### Low priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 23 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | fabric |
| 24 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | ide, fabric |
| 25 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | loom, stencil |
| 26 | **Discussion advisors: operator-named areas per advisor on top of the landed all-advisors default**<br>• Each discussion_advisors entry takes an optional area (a lens, as in review fans); without one the advisor answers every question | [`discussion-advisor-mode`](design-discussion-advisor-mode.md) | loom, stencil, config |
| 27 | **Fabric audit: plain readers accept unquoted globs and any flag**<br>• readOnlyCommands readers (cat, grep -r) accept unquoted globs and arbitrary flags | [`audit-reader-globs`](design-audit-reader-globs.md) | webster, fabric |

### Features

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Runs overview pane under the orch**<br>• A minimal pane under the orch listing every run in flight and its shed state; the per-run batten windows stay but the status bar hides them | [`runs-overview`](design-runs-overview.md) | reed, orch, batten |

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 2 | **An MCP server over lyx exposing the verbs agents need**<br>• A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes | [`lyx-mcp`](design-lyx-mcp.md) | shuttle, stencil |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | reed |
| 4 | **Kick-start pack: pre-resolved glyph spans for implementer and Burler**<br>• Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled | [`kick-start-pack`](design-kick-start-pack.md) | webster, quarry |
| 5 | **Orch loop: one land command and orch start --continue**<br>• The landing steps become one lyx command, and the orch resumes its last conversation after a reboot | [`orch-owns-the-loop`](design-orch-owns-the-loop.md) | batten, orch |
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
| 17 | **config: repo-wide default plus per-worktree override**<br>• A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works | [`config-overlay`](design-config-overlay.md) | config |
| 18 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | fabric |
| 19 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | config, board |
| 20 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | shuttle |
| 21 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | board, ide |
| 22 | **Shortcut commands built into lyx from one committed file**<br>• An embedded shortcut file, git-alias style: attach for reed attach, and one shortcut that starts the orch and attaches in the prime | [`operator-shortcuts`](design-operator-shortcuts.md) | orch, reed |
| 23 | **Open an extra named agent strand through shuttle**<br>• Open a named Claude session in a strand with no task, through shuttle, so it keeps its name; for extra strands in the prime | [`extra-agent-strand`](design-extra-agent-strand.md) | shuttle |
| 24 | **Who in a running task receives an instruction**<br>• Driver forwarding, an alias for the active strand kept by loom, or an on-call LLM per run: open for a brainstorm | [`run-message-routing`](design-run-message-routing.md) | loom |
| 25 | **A pane listing every gate of a run as a checklist with timings**<br>• Every gate of the run with a tick box and its elapsed time, in a pane of its own; an idea, not a priority | [`gate-checklist-pane`](design-gate-checklist-pane.md) | reed |
| 26 | **lyx cleanup: one command runs every module cleanup**<br>• Drop finished batten statuses, prune the board, clear disposable .scratch and .lyx files, each through the owning module verb | [`lyx-cleanup`](design-lyx-cleanup.md) | batten, board |
| 27 | **lyx pause: stop every run at once, safely and on purpose**<br>• A hub-wide pause that interrupts every Claude session with Esc within seconds, unlike the step-boundary pauses, recorded as ordered so no failure or idle path fires | [`lyx-halt`](design-lyx-halt.md) | batten |
| 28 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | reed, shuttle |
| 29 | **Weaver: a cold-start implementer beside Webster, with model escalation**<br>• A second implementer built on Millhouse Builder: a fresh agent per batch with its own model, escalating the model when an agent fails; Webster stays fork-based | [`weaver`](design-weaver.md) | webster |

#### Low priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 30 | **Plan-Write on MultiLLMProducer: advisors gather facts and check cards**<br>• Later: Plan-Write as the chair, with an advisor that maps callers, tests and verify commands per package, and possibly one that checks each card as it is written | [`plan-multi-llm`](design-plan-multi-llm.md) | loom, stencil |
