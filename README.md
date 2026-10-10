# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

#### Normal priority

| # | Task | Slug | At | Labels |
| --- | --- | --- | --- | --- |
| 1 | **Lighter test suite: measured tiers, cheap tmux fixtures, no lingering children**<br>• Measure each tier, non-login shells and shared servers in tier 3, lyx built once, no background gc in fixtures (#472), a no-tmux scan for tiers 1-2 | [`test-suite-weight`](design-test-suite-weight.md) | PR-Gate | shuttle, reed, fabric |
| 2 | **Webster and round-gate bug sweep: replans, batch checks, fork sandbox, shell wait, test re-runs**<br>• Webster stays consistent across replans, its batch and fabric checks judge only what they should, implementer forks stay sandboxed, the shell wait leaves nothing running, and the round gate re-runs whole tests | [`webster-replan-state`](design-webster-replan-state.md) | Webster | webster, loom, burler, fabric |
| 3 | **CLI surface: a command index for the orch, one help form, intake keeps comments**<br>• lyx help index injected into the orch, a test-enforced form for Use/Short/Long including board payload keys, and intake that carries issue comments | [`cli-surface`](design-cli-surface.md) | Plan-Burler | orch, board, selfreport |

### Ready

Waits on nothing open; can start now.

#### High priority

| # | Task | Slug | After | Labels |
| --- | --- | --- | --- | --- |
| 1 | **Darn recipe: one producer, then the usual PR; shared Publish, Finalize and teardown fixes**<br>• The darn recipe: a simplified run for small changes (one producer, verify retries to a budget, PR-Gate as usual), plus per-attempt verify logs, Finalize ignoring the parent weft batten records, teardown clearing abandoned claims, and Reflect reading the parent weft | [`quick-recipe`](design-quick-recipe.md) |  | loom, shed, batten, fabric, board |
| 2 | **Lens fan: master pre-reads the cited code before forking, forks get a read budget**<br>• Make the lens fan work as designed: the master reads the cited code (and judges nothing) before forking, each fork has a tool-call budget, and Discussion and Plan use their own fans | [`fan-master-preread`](design-fan-master-preread.md) |  | burler, stencil |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Batten idle notices count machine sleep as idle time**<br>• After a hibernate every run raises an idle notice for the whole sleep; the idle clock should skip time the machine was asleep | [`idle-notice-after-sleep`](design-idle-notice-after-sleep.md) | batten |
| 2 | **shuttle raises a hold notice at the skill-load turn end, before the start prompt is sent** | [`hold-on-skill-load`](design-hold-on-skill-load.md) | shuttle |
| 3 | **daemon idle Debug spawn lines reach no log sink; tmux socket path can exceed 108 bytes** | [`daemon-log-and-socket-path`](design-daemon-log-and-socket-path.md) | reed |
| 4 | **Build-tagged member declared once per tag set resolves ambiguous, so its plan Create can never pass record-batch** | [`quarry-build-tag-ambiguous`](design-quarry-build-tag-ambiguous.md) | webster, loom, quarry |
| 5 | **Bounce budget blocks a review segment after its round was approved, before the Bouncer verdict** | [`bounce-budget-after-approved`](design-bounce-budget-after-approved.md) | burler, loom |
| 6 | **Master's wait stops waiting on a still-running background shell at a turn end**<br>• A gate run still in a background shell is cut off at a turn end and the run still reports done | [`background-gate-shell-discarded`](design-background-gate-shell-discarded.md) | webster, shuttle |
| 7 | **Webster Master fixes a verify failure itself instead of spawning the fixer fork**<br>• Master edited and committed a test fix itself; the run-exit audit blocked it as a parent-write | [`master-self-fix`](design-master-self-fix.md) | webster, stencil |

### Enhancements

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Batten opens a terminal window per run instead of VS Code**<br>• VS Code per run is gone (c0d604f12); batten itself still opens no reed attach window, the orch does it by hand | [`ide-opt-in`](design-ide-opt-in.md) | ide, reed, batten |
| 2 | **Compare fanned and solo review rounds from their usage records**<br>• A report over round usage records for the fan-on decision; a lost usage record shows in the facts | [`fan-usage-report`](design-fan-usage-report.md) | burler |
| 3 | **Haiku 5.5 for the Bouncer judge: is its ruling simple enough**<br>• The driver already runs on Haiku 5.5; decide per Bouncer row whether the judge ruling is simple enough for Haiku, trial it, and switch only where it holds | [`haiku-roles`](design-haiku-roles.md) | config, loom, burler |
| 4 | **Burler runs its round on MultiLLMProducer**<br>• The fixer becomes the chair and the reviewer advisor-1 in one seat table, replacing roundHalves and await-review | [`burler-multi-llm`](design-burler-multi-llm.md) | burler, shed, stencil |
| 5 | **PATTERN overview breaks its cap only after merging main** | [`pattern-overview-headroom`](design-pattern-overview-headroom.md) | loom |
| 6 | **Reed watchdog children still burn idle CPU** | [`watchdog-child-cpu`](design-watchdog-child-cpu.md) | reed |
| 7 | **Bench the chair-and-advisors Discussion against the solo writer before switching it on**<br>• Same tasks through both producers, compare review rounds, findings, tokens and time; the operator watches a live run before it goes on | [`discussion-multi-llm-bench`](design-discussion-multi-llm-bench.md) | loom, stencil |

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 8 | **Convergence: log hub interventions and driver repairs per run by class**<br>• Record every hub intervention and driver repair with a class and cause, so interventions per run and repeat failure classes are visible | [`convergence-plan`](design-convergence-plan.md) | orch, shed |
| 9 | **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke**<br>• Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run | [`parent-review-hardening`](design-parent-review-hardening.md) | orch |
| 10 | **Reopen the prime VS Code window with one short command**<br>• A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root | [`ide-reopen-prime`](design-ide-reopen-prime.md) | ide |
| 11 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | fabric, loom |
| 12 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | shed |
| 13 | **lyx seeds the prime VS Code setup: title, orch start task, keybindings**<br>• LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json | [`ide-prime-setup`](design-ide-prime-setup.md) | ide, orch |
| 14 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | loom |
| 15 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | shed |
| 16 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | quarry |
| 17 | **Audit how plans use glyphs in practice, then tune the plan stencil**<br>• First audit done (findings below); re-run it by one agent after a good number of runs land on the implementer quarry hint (a28e79d9c) | [`glyph-practice-audit`](design-glyph-practice-audit.md) | loom, stencil |
| 18 | **Capture real hook payloads into fixtures**<br>• Run the llm-tier hook-capture scenario once and turn the real StopFailure, Notification and SessionEnd payloads into corpus fixtures | [`shuttle-hook-payload-corpus`](design-shuttle-hook-payload-corpus.md) | shuttle |
| 19 | **Shuttle: the launch line carries the first turn**<br>• /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed | [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) | shuttle |
| 20 | **Carry-over check re-reads the written file**<br>• WriteCarryOver checks the splice on the produced lines before the atomic rename; re-reading the file after the rename and checking what is on disk would also catch a concurrent writer | `carryover-verify-on-disk` | loom |
| 21 | **reed: strands state their size; parallel agents share the window evenly**<br>• Replace the one collapse-all-but-the-newest rule with per-strand sizing: content-sized panes like the runs overview, and parallel agents that share space evenly | [`reed-layout-shares`](design-reed-layout-shares.md) | reed |
| 22 | **Configurable file log level in logger.yaml**<br>• Trace files carry only high-level messages and logger.yaml sets no level; add a file log level so most of lyx can be traced on demand | [`logger-file-level`](design-logger-file-level.md) | config |
| 23 | **One owner for child-process lifetime in internal/proc**<br>• Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill | [`proc-child-lifetime`](design-proc-child-lifetime.md) |  |
| 24 | **Fold loomcli and orchcli tmux probe copies onto tmuxkit** | [`tmuxkit-probe-copies`](design-tmuxkit-probe-copies.md) | loom, orch, reed |
| 25 | **Reconcile keeps an old default below the new floor** | [`reconcile-stale-default`](design-reconcile-stale-default.md) | config |
| 26 | **Discussion advisors: operator-named areas per advisor on top of the landed all-advisors default**<br>• Each discussion_advisors entry takes an optional area (a lens, as in review fans); without one the advisor answers every question | [`discussion-advisor-mode`](design-discussion-advisor-mode.md) | loom, stencil, config |
| 27 | **Plan format has no way to insert a card or remove a card file** | [`plan-card-insert`](design-plan-card-insert.md) | loom, burler, stencil |
| 28 | **Reed strand panes still read ~/.tmux.conf and start login shells**<br>• test-suite-weight moved the test kit to non-login panes without the user tmux config; reed own servers still read ~/.tmux.conf and start login shells for strand panes, noted as a bound in PATTERN | `reed-strand-login-shells` | reed |
| 29 | **Re-measure the test suite weight on a quiet machine**<br>• The test-suite-weight before and after numbers were taken under load; rerun the report Regenerate section on an idle machine for a fair wall-time comparison | `suite-weight-quiet-rerun` |  |

#### Low priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 30 | **Fabric's Windows path behaviour verified on a Windows host**<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | fabric |
| 31 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history**<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | ide, fabric |

### Features

#### High priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Runs overview pane under the orch; batten windows hidden**<br>• A minimal pane under the orch listing every run in flight and its shed state; the per-run batten windows stay but the status bar hides them | [`runs-overview`](design-runs-overview.md) | reed, orch, batten |

#### Normal priority

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 2 | **An MCP server over lyx exposing the verbs agents need**<br>• A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes | [`lyx-mcp`](design-lyx-mcp.md) | shuttle, stencil |
| 3 | **creel: the one way to reach an agent**<br>• Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages | [`creel`](design-creel.md) | reed |
| 4 | **Kick-start pack: pre-resolved glyph spans for implementer and Burler**<br>• Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled | [`kick-start-pack`](design-kick-start-pack.md) | webster, quarry |
| 5 | **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt**<br>• Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot | [`orch-owns-the-loop`](design-orch-owns-the-loop.md) | batten, orch |
| 6 | **Hardener: an autonomous behavior-review campaign that replaces crucible**<br>• Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges | [`hardener`](design-hardener.md) | burler |
| 7 | **Orch and batten run from the operator's own long-lived worktree, not only the prime**<br>• Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch | [`orch-any-worktree`](design-orch-any-worktree.md) | fabric, batten, orch |
| 8 | **lyx ide open: launch VS Code without writing anything**<br>• spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb | [`ide-open-verb`](design-ide-open-verb.md) | ide, batten |
| 9 | **Pairs as folders in the prime's hub workspace**<br>• Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays | [`ide-workspace-pairs`](design-ide-workspace-pairs.md) | ide, fabric |
| 10 | **semantic-index: find code by concept, not by literal keyword**<br>• Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep | [`semantic-index`](design-semantic-index.md) | quarry |
| 11 | **Ship lyx as an installable Claude Code plugin**<br>• Package lyx and its skills as one installable plugin | [`plugin-packaging`](design-plugin-packaging.md) | stencil |
| 12 | **Codex as a second agent engine**<br>• If lyx gets a second engine, Codex is more relevant than Gemini | [`codex-engine`](design-codex-engine.md) | shuttle |
| 13 | **Fabric's ordinary-monorepo verb surface**<br>• Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame | [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) | fabric |
| 14 | **Talk to the orch through Slack**<br>• A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed | [`reed-slack-relay`](design-reed-slack-relay.md) | reed |
| 15 | **raddle: codeguide's woven-in successor**<br>• Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist | [`raddle`](design-raddle.md) | loom |
| 16 | **webster: worktree-per-card parallel execution**<br>• Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays | [`webster-parallel-execution`](design-webster-parallel-execution.md) | webster |
| 17 | **reed: cross-worktree columns, own-window anchoring, per-window attach**<br>• Three related ideas for reed beyond one pane per strand and one window per worktree | [`reed-multi-window`](design-reed-multi-window.md) | reed |
| 18 | **Baseline producer and numerical-drift gate**<br>• Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift | [`baseline-drift-gate`](design-baseline-drift-gate.md) | webster, loom |
| 19 | **config: repo-wide default plus per-worktree override**<br>• A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works | [`config-overlay`](design-config-overlay.md) | config |
| 20 | **fabric: two-sided reset-to-SHA verb**<br>• The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides | [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) | fabric |
| 21 | **doctor: a lyx diagnostics command**<br>• lyx doctor checks the _lyx layout, config parse, board reachability and stale locks | [`doctor`](design-doctor.md) | config, board |
| 22 | **session sync: Claude transcripts across machines**<br>• Copy Claude .jsonl transcripts across machines so --resume works elsewhere | [`session-sync`](design-session-sync.md) | shuttle |
| 23 | **Interactive board view: a regenerated HTML file and a phone-friendly README**<br>• board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app | [`board-viewer`](design-board-viewer.md) | board, ide |
| 24 | **Shortcut commands built into lyx from one committed file**<br>• An embedded shortcut file, git-alias style: attach for reed attach, and one shortcut that starts the orch and attaches in the prime | [`operator-shortcuts`](design-operator-shortcuts.md) | orch, reed |
| 25 | **Open an extra named agent strand through shuttle**<br>• Open a named Claude session in a strand with no task, through shuttle, so it keeps its name; for extra strands in the prime | [`extra-agent-strand`](design-extra-agent-strand.md) | shuttle |
| 26 | **Who in a running task receives an instruction**<br>• Driver forwarding, an alias for the active strand kept by loom, or an on-call LLM per run: open for a brainstorm | [`run-message-routing`](design-run-message-routing.md) | loom |
| 27 | **A pane listing every gate of a run as a checklist with timings**<br>• Every gate of the run with a tick box and its elapsed time, in a pane of its own; an idea, not a priority | [`gate-checklist-pane`](design-gate-checklist-pane.md) | reed |
| 28 | **lyx cleanup: one command runs every module cleanup**<br>• Drop finished batten statuses, prune the board, clear disposable .scratch and .lyx files, each through the owning module verb | [`lyx-cleanup`](design-lyx-cleanup.md) | batten, board |
| 29 | **lyx pause: stop every run at once, safely and on purpose**<br>• A hub-wide pause that interrupts every Claude session with Esc within seconds, unlike the step-boundary pauses, recorded as ordered so no failure or idle path fires | [`lyx-halt`](design-lyx-halt.md) | batten |
| 30 | **Spawn audit log: every process lyx starts is recorded and traceable**<br>• One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash | [`spawn-audit-log`](design-spawn-audit-log.md) | reed, shuttle |
| 31 | **Plan-Write on MultiLLMProducer: advisors gather facts and check cards**<br>• Later: Plan-Write as the chair, with an advisor that maps callers, tests and verify commands per package, and possibly one that checks each card as it is written | [`plan-multi-llm`](design-plan-multi-llm.md) | loom, stencil |

## Done

Finished, awaiting `lyx board prune`.

| # | Entry | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Driver steps in only on trouble: Go loops the steps, context under 100k**<br>• Go runs the shed steps and wakes the driver only at a stop, so a Haiku driver stays under 100k; the driver can still stop a run at a point the orch names | [`driver-context`](design-driver-context.md) | shed, stencil, loom |
