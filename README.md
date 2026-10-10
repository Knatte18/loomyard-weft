# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | At | Labels |
| --- | --- | --- | --- | --- |
| 1 | **Lighter test suite: measured tiers, cheap tmux fixtures, no lingering children**<br>• Measure each tier, non-login shells and shared servers in tier 3, lyx built once, no background gc in fixtures (#472), a no-tmux scan for tiers 1-2 | [`test-suite-weight`](design-test-suite-weight.md) | Webster | shuttle, reed, fabric |
| 2 | **Discussion-Write on MultiLLMProducer: a chair writes, advisors answer**<br>• Design and build a Discussion-Write whose writer is the chair, with advisors that study the code and answer its design questions live | [`discussion-multi-llm`](design-discussion-multi-llm.md) | Webster | loom, stencil, shed |
| 3 | **Driver steps in only on trouble: Go loops the steps, context under 100k**<br>• Go runs the shed steps and wakes the driver only at a stop, so a Haiku driver stays under 100k; the driver can still stop a run at a point the orch names | [`driver-context`](design-driver-context.md) | Plan-Burler | shed, stencil, loom |

### Ready

Waits on nothing open; can start now.

| # | Task | Slug | After | Labels |
| --- | --- | --- | --- | --- |
| 1 | **Webster and round-gate bug sweep: replans, batch checks, fork sandbox, shell wait, test re-runs**<br>• Webster stays consistent across replans, its batch and fabric checks judge only what they should, implementer forks stay sandboxed, the shell wait leaves nothing running, and the round gate re-runs whole tests | [`webster-replan-state`](design-webster-replan-state.md) |  | webster, loom, burler, fabric |
| 2 | **Burler runs its round on MultiLLMProducer**<br>• The fixer becomes the chair and the reviewer advisor-1 in one seat table, replacing roundHalves and await-review | [`burler-multi-llm`](design-burler-multi-llm.md) |  | burler, shed, stencil |
| 3 | **CLI surface: a command index for the orch, one help form, intake keeps comments**<br>• lyx help index injected into the orch, a test-enforced form for Use/Short/Long including board payload keys, and intake that carries issue comments | [`cli-surface`](design-cli-surface.md) |  | orch, board, selfreport |

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Batten idle notices count machine sleep as idle time**<br>• After a hibernate every run raises an idle notice for the whole sleep; the idle clock should skip time the machine was asleep | [`idle-notice-after-sleep`](design-idle-notice-after-sleep.md) | batten |
| 2 | **shuttle raises a hold notice at the skill-load turn end, before the start prompt is sent** | [`hold-on-skill-load`](design-hold-on-skill-load.md) | shuttle |
| 3 | **daemon idle Debug spawn lines reach no log sink; tmux socket path can exceed 108 bytes** | [`daemon-log-and-socket-path`](design-daemon-log-and-socket-path.md) | reed |
| 4 | **Build-tagged member declared once per tag set resolves ambiguous, so its plan Create can never pass record-batch** | [`quarry-build-tag-ambiguous`](design-quarry-build-tag-ambiguous.md) | webster, loom |
| 5 | **Finalize blocks on the parent weft's uncommitted batten status records** | [`finalize-weft-batten-dirty`](design-finalize-weft-batten-dirty.md) | fabric, batten |
| 6 | **Publish verify log is overwritten by a later run, so the failure is lost** | [`verify-log-overwritten`](design-verify-log-overwritten.md) |  |

### Enhancements

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Batten opens a terminal window per run instead of VS Code** _(high priority)_<br>• VS Code per run is gone (c0d604f12); batten itself still opens no reed attach window, the orch does it by hand | [`ide-opt-in`](design-ide-opt-in.md) | ide, reed, batten |
| 2 | **Compare fanned and solo review rounds from their usage records** _(high priority)_<br>• A report over round usage records for the fan-on decision; a lost usage record shows in the facts | [`fan-usage-report`](design-fan-usage-report.md) | burler |
| 3 | **Haiku 5.5 for the Bouncer judge: is its ruling simple enough** _(high priority)_<br>• The driver already runs on Haiku 5.5; decide per Bouncer row whether the judge ruling is simple enough for Haiku, trial it, and switch only where it holds | [`haiku-roles`](design-haiku-roles.md) | config, loom, burler |
| 4 | **An abandoned run leaves its board entry locked** _(high priority)_<br>• Batten teardown clears the run status of a run that ends without landing, so the entry is editable again | [`abandoned-run-claim`](design-abandoned-run-claim.md) | batten, board |
| 5 | **PATTERN overview breaks its cap only after merging main** _(high priority)_ | [`pattern-overview-headroom`](design-pattern-overview-headroom.md) | loom |
| 6 | **Reed watchdog children still burn idle CPU** _(high priority)_ | [`watchdog-child-cpu`](design-watchdog-child-cpu.md) | reed |
| 7 | **Convergence: log hub interventions and driver repairs per run by class**<br>• Record every hub intervention and driver repair with a class and cause, so interventions per run and repeat failure classes are visible | [`convergence-plan`](design-convergence-plan.md) | orch, shed |
| 8 | **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke**<br>• Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run | [`parent-review-hardening`](design-parent-review-hardening.md) | orch |
| 9 | **Reopen the prime VS Code window with one short command**<br>• A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root | [`ide-reopen-prime`](design-ide-reopen-prime.md) | ide |
| 10 | **finalize: the discrepancy-document conflict shape**<br>• Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers | [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) | fabric, loom |
| 11 | **shedrecipe: capability declaration instead of manual seam-threading**<br>• A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer | [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) | shed |
| 12 | **lyx seeds the prime VS Code setup: title, orch start task, keybindings**<br>• LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json | [`ide-prime-setup`](design-ide-prime-setup.md) | ide, orch |
| 13 | **Discussion stage has no sanctioned path to land a repository edit ahead of Plan**<br>• Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan | [`discussion-lands-invariant`](design-discussion-lands-invariant.md) | loom |
| 14 | **Windows verify timeout kills the whole process tree**<br>• On Windows the verifytree timeout kills only the shell, not the process tree it started | [`verify-watchdog`](design-verify-watchdog.md) | shed |
| 15 | **Kick-start packs from tool locations through quarry Enclose**<br>• Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack | [`enclose-kickstart`](design-enclose-kickstart.md) | quarry |
| 16 | **Audit how plans use glyphs in practice, then tune the plan stencil**<br>• Runs only after many tasks have landed with glyph plans: measure glyph use, how writers find glyphs, Plan rounds and handle binding from real runs; kick-start waits on it | [`glyph-practice-audit`](design-glyph-practice-audit.md) | loom, stencil |
| 17 | **Capture real hook payloads into fixtures**<br>• Run the llm-tier hook-capture scenario once and turn the real StopFailure, Notification and SessionEnd payloads into corpus fixtures | [`shuttle-hook-payload-corpus`](design-shuttle-hook-payload-corpus.md) | shuttle |
| 18 | **Shuttle: the launch line carries the first turn**<br>• /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed | [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) | shuttle |
| 19 | **Carry-over check re-reads the written file**<br>• WriteCarryOver checks the splice on the produced lines before the atomic rename; re-reading the file after the rename and checking what is on disk would also catch a concurrent writer | `carryover-verify-on-disk` | loom |
| 20 | **reed: strands state their size; parallel agents share the window evenly**<br>• Replace the one collapse-all-but-the-newest rule with per-strand sizing: content-sized panes like the runs overview, and parallel agents that share space evenly | [`reed-layout-shares`](design-reed-layout-shares.md) | reed |
| 21 | **Configurable file log level in logger.yaml**<br>• Trace files carry only high-level messages and logger.yaml sets no level; add a file log level so most of lyx can be traced on demand | [`logger-file-level`](design-logger-file-level.md) | config |
| 22 | **One owner for child-process lifetime in internal/proc**<br>• Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill | [`proc-child-lifetime`](design-proc-child-lifetime.md) |  |
| 23 | **Fold loomcli and orchcli tmux probe copies onto tmuxkit** | [`tmuxkit-probe-copies`](design-tmuxkit-probe-copies.md) | loom, orch, reed |
| 24 | **Reconcile keeps an old default below the new floor** | [`reconcile-stale-default`](design-reconcile-stale-default.md) | config |
| 25 | **Fabric's Windows path behaviour verified on a Windows host** _(low priority)_<br>• Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant | [`fabric-windows`](design-fabric-windows.md) | fabric |
| 26 | **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** _(low priority)_<br>• Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date | [`warp-visibility`](design-warp-visibility.md) | ide, fabric |

### Features

| # | Note | Slug | Labels |
| --- | --- | --- | --- |
| 1 | **Runs overview pane under the orch; batten windows hidden** _(high priority)_<br>• A minimal pane under the orch listing every run in flight and its shed state; the per-run batten windows stay but the status bar hides them | [`runs-overview`](design-runs-overview.md) | reed, orch, batten |
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
| 1 | **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically**<br>• Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first | [`plan-glyph-precision`](design-plan-glyph-precision.md) | loom, quarry |
| 2 | **board: refuse scope edits to a running task**<br>• The board already marks running tasks; a write that changes the scope of a running task is refused | [`board-lock-running`](design-board-lock-running.md) | board |
| 3 | **Orch and batten: compact reload, one event-driven notice channel, a window per batten**<br>• The orch reloads as soon as compaction lands, batten sends one timely notice per state change without ticks, and each batten gets its own tmux window | [`lifecycle-waits`](design-lifecycle-waits.md) | orch, batten |
| 4 | **Fabric: pair create survives a refused push, no untested main from the prime, neutral export names**<br>• Retry a refused weft push and roll back the own branch, gate the prime-side push of main, rename fabric exports that carry warp/weft | [`fabric-pushes`](design-fabric-pushes.md) | fabric |
| 5 | **Shuttle: one session-state model, held turns as waits, stuck not a question, a deliberate permission setup**<br>• Shuttle computes busy/idle-done/idle-stalled/asking/dead from hooks in shadow mode, holds show as waits, a stuck outcome is classified from its file, and the hook denies python | [`shuttle-state`](design-shuttle-state.md) | shuttle |
| 6 | **reed: strand and run tabs, narrow-terminal run row, a color per strand role**<br>• VIEW plus a zoom button per strand and Alt-key navigation, run tabs that never fall off a narrow terminal, and each strand role in its own color | [`reed-view`](design-reed-view.md) | reed |
| 7 | **Shuttle: a Send lands or retries, and attach knows a half it stopped**<br>• Send waits for a ready pane and a settled input before Enter, retries with backoff, and a lost submission is transient; attach no longer halts on a stopped Burler half | [`shuttle-send`](design-shuttle-send.md) | shuttle |
| 8 | **Webster: recovery never dead-ends, stale config fails at the start, a computed Merriam base**<br>• Reset targets as lyx verbs, a recovery prompt that names a dirty tree, decisions that reach a begun card, a retryable failed record-batch, batcher.yaml checked in preflight, and a computed batchifier base | [`webster-dead-ends`](design-webster-dead-ends.md) | webster |
| 9 | **Review loop: a lighter convergence rule, a three-round default budget, models per segment**<br>• Converge without circling to the budget on fresh edge-case keys, default the round budget to 3, set review and fix models per segment, an option to start the fixer after the review, and circling decisions that resume the run | [`review-loop-convergence`](design-review-loop-convergence.md) | burler, loom |
| 10 | **Review lens fans for Discussion and Plan, narrowed by the judge each round**<br>• Wire burler cluster review into Discussion and Plan with their own fans, let the judge drop settled lenses, after shuttle-state settles gate quiescence with forks | [`review-lens-fan`](design-review-lens-fan.md) | burler, loom |
| 11 | **A hub survives a lyx binary change; the no-script rule reaches spawned agents**<br>• Config reconciles itself on the first lyx command after a binary change, stencil drift behind main stops warning source-ahead, and file-editing stencils state the no-script edit rule | [`binary-change-and-rules`](design-binary-change-and-rules.md) | config, stencil |
| 12 | **Plan gate checks: precise callers, nested fixture modules, a reviewer that knows the gate**<br>• Resolve callers by import path and receiver type, let gates run against nested fixture modules, and give the plan reviewer the gate check list | [`plan-gate-checks`](design-plan-gate-checks.md) | loom, quarry |
| 13 | **Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base**<br>• Accept a prefix head_sha, find forked batch transcripts, accept a Card-Index-only change on an older record, and re-measure the Merriam fixed context | [`webster-record-batch`](design-webster-record-batch.md) | webster |
| 14 | **Gate test scope: which gate runs which tier, bounded hub-wide**<br>• Card gate tests only its own packages at tier 1, tier 2 per batch and at the plan verify, a hub-wide slot lock for gate test runs, guarded publish_verify | [`gate-test-scope`](design-gate-test-scope.md) | webster, loom, stencil, config, burler |
| 15 | **Git robustness: stale pack index, background gc, network timeouts, reused slugs**<br>• Reindex-and-retry for every go-git call, no auto-gc under a live step, timeouts on git network calls, batten refuses a done slug, bounded HeadContains | [`git-robustness`](design-git-robustness.md) | fabric, batten, stencil |
| 16 | **Publish and loom recovery: a fix route from Publish, resume past its own merge, clean re-steps**<br>• A blocked Publish hands its failure to a fixer, resume survives its own staged merge, no false config warning at Finalize, re-step and decision-author fixes | [`publish-recovery`](design-publish-recovery.md) | loom, burler, fabric, config, stencil |
| 17 | **MultiLLMProducer: one producer runs a chair and its advisors as seats**<br>• A producer beside SingleLLMProducer that runs N agents as seats, one chair that steers and decides the outcome plus advisor-1..n, with its own internal message channel; built standalone, used by nothing yet | [`multi-llm-producer`](design-multi-llm-producer.md) | shed, shuttle |
| 18 | **Daemons and wake-ups: event-driven waits, a timely orch reload, recorded revival order**<br>• A rule that lyx daemons wait on events or second-scale polls (reed watchdog first), the orch watcher reloads right after compaction, reed revives sessions by recorded start time | [`daemon-wakeups`](design-daemon-wakeups.md) | reed, orch, shuttle |
| 19 | **Bolt and hub store: seeding without ping-pong, reconcile on a fresh Bolt, gc that stays out of the way**<br>• Seed stencils forward only and from a pulled Bolt, report failed reconcile pushes, and keep the hub store gc off busy runs and repeat repacks | [`bolt-and-store`](design-bolt-and-store.md) | stencil, fabric, config |
