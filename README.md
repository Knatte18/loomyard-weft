# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Running

Held by a run; its scope is locked until the run ends.

1. **Gate test scope: which gate runs which tier, bounded hub-wide** — [`gate-test-scope`](design-gate-test-scope.md) · bug, enhancement, webster, loom, stencil, config, burler · running · Plan-Burler
   - Card gate tests only its own packages at tier 1, tier 2 per batch and at the plan verify, a hub-wide slot lock for gate test runs, guarded publish_verify
   - **Before:** `test-suite-weight`, `publish-recovery`, `burler-multi-llm`, `discussion-multi-llm`, `cli-surface`, `webster-replan-state`
1. **Git robustness: stale pack index, background gc, network timeouts, reused slugs** — [`git-robustness`](design-git-robustness.md) · bug, enhancement, fabric, batten, stencil · running · Webster-Burler
   - Reindex-and-retry for every go-git call, no auto-gc under a live step, timeouts on git network calls, batten refuses a done slug, bounded HeadContains
   - **Before:** `publish-recovery`, `cli-surface`
1. **MultiLLMProducer: one producer runs a chair and its advisors as seats** — [`multi-llm-producer`](design-multi-llm-producer.md) · feature, shed, shuttle · running · Discussion-Burler
   - A producer beside SingleLLMProducer that runs N agents as seats, one chair that steers and decides the outcome plus advisor-1..n, with its own internal message channel; built standalone, used by nothing yet
   - **Before:** `burler-multi-llm`, `discussion-multi-llm`

### Layer A

Waits on nothing that is not running; next to start.

1. **Lighter test suite: measured tiers, cheap tmux fixtures, no lingering children** — [`test-suite-weight`](design-test-suite-weight.md) · bug, enhancement, shuttle, reed, fabric
   - Measure each tier, non-login shells and shared servers in tier 3, lyx built once, no background gc in fixtures (#472), a no-tmux scan for tiers 1-2
   - **After:** `gate-test-scope`
1. **Publish and loom recovery: a fix route from Publish, resume past its own merge, clean re-steps** — [`publish-recovery`](design-publish-recovery.md) · bug, loom, burler, fabric, config, stencil
   - A blocked Publish hands its failure to a fixer, resume survives its own staged merge, no false config warning at Finalize, re-step and decision-author fixes
   - **After:** `gate-test-scope`, `git-robustness`
   - **Before:** `webster-replan-state`
1. **Burler runs its round on MultiLLMProducer** — [`burler-multi-llm`](design-burler-multi-llm.md) · enhancement, burler, shed, stencil
   - The fixer becomes the chair and the reviewer advisor-1 in one seat table, replacing roundHalves and await-review
   - **After:** `multi-llm-producer`, `gate-test-scope`
1. **Discussion-Write on MultiLLMProducer: a chair writes, advisors answer** — [`discussion-multi-llm`](design-discussion-multi-llm.md) · feature, loom, stencil, shed
   - Design and build a Discussion-Write whose writer is the chair, with advisors that study the code and answer its design questions live
   - **After:** `multi-llm-producer`, `gate-test-scope`
1. **CLI surface: a command index for the orch, one help form, intake keeps comments** — [`cli-surface`](design-cli-surface.md) · bug, enhancement, orch, board, selfreport
   - lyx help index injected into the orch, a test-enforced form for Use/Short/Long including board payload keys, and intake that carries issue comments
   - **After:** `gate-test-scope`, `git-robustness`
1. **Daemons and wake-ups: event-driven waits, a timely orch reload, recorded revival order** — [`daemon-wakeups`](design-daemon-wakeups.md) · bug, enhancement, reed, orch, shuttle
   - A rule that lyx daemons wait on events or second-scale polls (reed watchdog first), the orch watcher reloads right after compaction, reed revives sessions by recorded start time

### Layer B

Starts when every entry it names under After is done.

1. **Webster state across replans: goto to Plan, late plan reviews, rebaseline clarity** — [`webster-replan-state`](design-webster-replan-state.md) · bug, enhancement, webster, loom, burler
   - Webster stays consistent after a goto to Plan or a late Plan-Review round, rebaseline names its fallback, one-call head_sha check, verify-module splits at || and |
   - **After:** `gate-test-scope`, `publish-recovery`

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

1. **Batten idle notices count machine sleep as idle time** — [`idle-notice-after-sleep`](design-idle-notice-after-sleep.md) · bug, batten
   - After a hibernate every run raises an idle notice for the whole sleep; the idle clock should skip time the machine was asleep
1. **Stencil seeding ping-pongs on Bolt: prod-mode binaries overwrite each other's stencils** — [`stencil-seed-ping-pong`](design-stencil-seed-ping-pong.md) · bug, stencil
1. **fabric reconcile on a stale Bolt: seed commits conflict on push, yet ok:true** — [`reconcile-stale-bolt`](design-reconcile-stale-bolt.md) · bug, fabric, config

### Enhancements

1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · enhancement, orch, shed
   - Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · enhancement, orch
   - Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **Reopen the prime VS Code window with one short command** — [`ide-reopen-prime`](design-ide-reopen-prime.md) · enhancement, ide
   - A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root
1. **Fabric's Windows path behaviour verified on a Windows host** — [`fabric-windows`](design-fabric-windows.md) · enhancement, fabric
   - Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant
1. **Batten opens a terminal window per run instead of VS Code** — [`ide-opt-in`](design-ide-opt-in.md) · enhancement, ide, reed, batten
   - VS Code per run is gone (c0d604f12); batten itself still opens no reed attach window, the orch does it by hand
1. **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** — [`warp-visibility`](design-warp-visibility.md) · enhancement, ide, fabric
   - Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date
1. **finalize: the discrepancy-document conflict shape** — [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) · enhancement, fabric, loom
   - Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers
1. **shedrecipe: capability declaration instead of manual seam-threading** — [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) · enhancement, shed
   - A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer
1. **lyx seeds the prime VS Code setup: title, orch start task, keybindings** — [`ide-prime-setup`](design-ide-prime-setup.md) · enhancement, ide, orch
   - LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json
1. **Discussion stage has no sanctioned path to land a repository edit ahead of Plan** — [`discussion-lands-invariant`](design-discussion-lands-invariant.md) · enhancement, loom
   - Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan
1. **Step-level watchdog and a Windows verify timeout that kills the process tree** — [`verify-watchdog`](design-verify-watchdog.md) · enhancement, shed
   - A lyx shed step that never returns is noticed, and the Windows verify timeout kills the whole process tree
1. **Kick-start packs from tool locations through quarry Enclose** — [`enclose-kickstart`](design-enclose-kickstart.md) · enhancement, quarry
   - Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack
1. **Audit how plans use glyphs in practice, then tune the plan stencil** — [`glyph-practice-audit`](design-glyph-practice-audit.md) · enhancement, loom, stencil
   - Runs only after many tasks have landed with glyph plans: measure glyph use, how writers find glyphs, Plan rounds and handle binding from real runs; kick-start waits on it
1. **Capture real hook payloads into fixtures** — [`shuttle-hook-payload-corpus`](design-shuttle-hook-payload-corpus.md) · enhancement, shuttle
   - Run the llm-tier hook-capture scenario once and turn the real StopFailure, Notification and SessionEnd payloads into corpus fixtures
1. **Shuttle: the launch line carries the first turn** — [`shuttle-color-verified-send`](design-shuttle-color-verified-send.md) · enhancement, shuttle
   - /color already goes through the verified send; the skill-load message or the prompt should ride the launch line so only later turns are typed
1. **Carry-over check re-reads the written file** — `carryover-verify-on-disk` · enhancement, loom
   - WriteCarryOver checks the splice on the produced lines before the atomic rename; re-reading the file after the rename and checking what is on disk would also catch a concurrent writer
1. **reed: strands state their size; parallel agents share the window evenly** — [`reed-layout-shares`](design-reed-layout-shares.md) · enhancement, reed
   - Replace the one collapse-all-but-the-newest rule with per-strand sizing: content-sized panes like the runs overview, and parallel agents that share space evenly
1. **Compare fanned and solo review rounds from their usage records** — [`fan-usage-report`](design-fan-usage-report.md) · enhancement, burler
   - A report over round usage records for the fan-on decision; a lost usage record shows in the facts
1. **Configurable file log level in logger.yaml** — [`logger-file-level`](design-logger-file-level.md) · enhancement, config
   - Trace files carry only high-level messages and logger.yaml sets no level; add a file log level so most of lyx can be traced on demand
1. **Haiku 5.5 for the light roles: driver and judge** — [`haiku-roles`](design-haiku-roles.md) · enhancement, config, loom, burler
   - Move the roles that only steer or rule on a fixed question, the loom driver and the Bouncer judge, to Haiku 5.5 where a trial run shows it holds
1. **One owner for child-process lifetime in internal/proc** — [`proc-child-lifetime`](design-proc-child-lifetime.md) · enhancement · 
   - Fold lyx gate test's process-group, Pdeathsig and job-object setup into internal/proc beside ConfigureGroupKill
1. **An abandoned run leaves its board entry locked** — [`abandoned-run-claim`](design-abandoned-run-claim.md) · enhancement, batten, board
   - Batten teardown clears the run status of a run that ends without landing, so the entry is editable again

### Features

1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature, shuttle, stencil
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: the one way to reach an agent** — [`creel`](design-creel.md) · feature, reed
   - Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages
1. **Runs overview pane under the orch; batten windows hidden** — [`runs-overview`](design-runs-overview.md) · feature, reed, orch, batten
   - A minimal pane under the orch listing every run in flight and its shed state; the per-run batten windows stay but the status bar hides them
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature, webster, quarry
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · feature, batten, orch
   - Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · feature, burler
   - Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges
1. **Orch and batten run from the operator's own long-lived worktree, not only the prime** — [`orch-any-worktree`](design-orch-any-worktree.md) · feature, fabric, batten, orch
   - Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch
1. **lyx ide open: launch VS Code without writing anything** — [`ide-open-verb`](design-ide-open-verb.md) · feature, ide, batten
   - spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb
1. **Pairs as folders in the prime's hub workspace** — [`ide-workspace-pairs`](design-ide-workspace-pairs.md) · feature, ide, fabric
   - Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays
1. **semantic-index: find code by concept, not by literal keyword** — [`semantic-index`](design-semantic-index.md) · feature, quarry
   - Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep
1. **Ship lyx as an installable Claude Code plugin** — [`plugin-packaging`](design-plugin-packaging.md) · feature, stencil
   - Package lyx and its skills as one installable plugin
1. **Codex as a second agent engine** — [`codex-engine`](design-codex-engine.md) · feature, shuttle
   - If lyx gets a second engine, Codex is more relevant than Gemini
1. **Fabric's ordinary-monorepo verb surface** — [`fabric-monorepo-verbs`](design-fabric-monorepo-verbs.md) · feature, fabric
   - Against plain git, fabric still lacks log, show, branch, tag, stash, revert, restore, rm/mv, rebase, cherry-pick and blame
1. **Talk to the orch through Slack** — [`reed-slack-relay`](design-reed-slack-relay.md) · feature, reed
   - A Slack relay so the operator can reach the orch from a phone where Claude's own app is not allowed
1. **raddle: codeguide's woven-in successor** — [`raddle`](design-raddle.md) · feature, loom
   - Docs regenerated once at merge time over the plan's diff, folded into Finalize's contract; the geometry and the parallel-regeneration design exist
1. **webster: worktree-per-card parallel execution** — [`webster-parallel-execution`](design-webster-parallel-execution.md) · feature, webster
   - Give each DAG-independent group its own fabric-spawned worktree so concurrent cards stop sharing one git index; parked until measured evidence says it pays
1. **reed: cross-worktree columns, own-window anchoring, per-window attach** — [`reed-multi-window`](design-reed-multi-window.md) · feature, reed
   - Three related ideas for reed beyond one pane per strand and one window per worktree
1. **Baseline producer and numerical-drift gate** — [`baseline-drift-gate`](design-baseline-drift-gate.md) · feature, webster, loom
   - Capture a before-state so a failure the task caused can be told from a pre-existing one; in numerical repos the failure to catch is drift
1. **config: repo-wide default plus per-worktree override** — [`config-overlay`](design-config-overlay.md) · feature, config
   - A repo-wide default layer read from _board with each worktree's own file as an override, the way millhouse's config.local.yaml works
1. **fabric: two-sided reset-to-SHA verb** — [`fabric-reset-to-sha`](design-fabric-reset-to-sha.md) · feature, fabric
   - The post-conclude undo the merge surface does not ship: reset to a warp SHA, resolve the paired weft SHA through the correspondence index, gate both sides
1. **doctor: a lyx diagnostics command** — [`doctor`](design-doctor.md) · feature, config, board
   - lyx doctor checks the _lyx layout, config parse, board reachability and stale locks
1. **session sync: Claude transcripts across machines** — [`session-sync`](design-session-sync.md) · feature, shuttle
   - Copy Claude .jsonl transcripts across machines so --resume works elsewhere
1. **Interactive board view: a regenerated HTML file and a phone-friendly README** — [`board-viewer`](design-board-viewer.md) · feature, board, ide
   - board.html rendered with README on every write; a VS Code task opens it in the browser; collapsible README for the GitHub app
1. **Shortcut commands built into lyx from one committed file** — [`operator-shortcuts`](design-operator-shortcuts.md) · feature, orch, reed
   - An embedded shortcut file, git-alias style: attach for reed attach, and one shortcut that starts the orch and attaches in the prime
1. **Open an extra named agent strand through shuttle** — [`extra-agent-strand`](design-extra-agent-strand.md) · feature, shuttle
   - Open a named Claude session in a strand with no task, through shuttle, so it keeps its name; for extra strands in the prime
1. **Who in a running task receives an instruction** — [`run-message-routing`](design-run-message-routing.md) · feature, loom
   - Driver forwarding, an alias for the active strand kept by loom, or an on-call LLM per run: open for a brainstorm
1. **A pane listing every gate of a run as a checklist with timings** — [`gate-checklist-pane`](design-gate-checklist-pane.md) · feature, reed
   - Every gate of the run with a tick box and its elapsed time, in a pane of its own; an idea, not a priority
1. **lyx cleanup: one command runs every module cleanup** — [`lyx-cleanup`](design-lyx-cleanup.md) · feature, batten, board
   - Drop finished batten statuses, prune the board, clear disposable .scratch and .lyx files, each through the owning module verb
1. **lyx pause: stop every run at once, safely and on purpose** — [`lyx-halt`](design-lyx-halt.md) · feature, batten
   - A hub-wide pause that interrupts every Claude session with Esc within seconds, unlike the step-boundary pauses, recorded as ordered so no failure or idle path fires
1. **Spawn audit log: every process lyx starts is recorded and traceable** — [`spawn-audit-log`](design-spawn-audit-log.md) · feature, reed, shuttle
   - One recorded launch path and a hub-wide spawn log with owner, argv and resource use, plus a sampler and a summary verb, so a process storm can be traced after a crash
1. **Plan-Write on MultiLLMProducer: advisors gather facts and check cards** — [`plan-multi-llm`](design-plan-multi-llm.md) · feature, loom, stencil
   - Later: Plan-Write as the chair, with an advisor that maps callers, tests and verify commands per package, and possibly one that checks each card as it is written

## Done

Finished, awaiting `lyx board prune`.

1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · enhancement, loom, quarry · done
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
1. **board: refuse scope edits to a running task** — [`board-lock-running`](design-board-lock-running.md) · bug, board · done
   - The board already marks running tasks; a write that changes the scope of a running task is refused
1. **Orch and batten: compact reload, one event-driven notice channel, a window per batten** — [`lifecycle-waits`](design-lifecycle-waits.md) · bug, enhancement, orch, batten · done
   - The orch reloads as soon as compaction lands, batten sends one timely notice per state change without ticks, and each batten gets its own tmux window
1. **Fabric: pair create survives a refused push, no untested main from the prime, neutral export names** — [`fabric-pushes`](design-fabric-pushes.md) · bug, enhancement, fabric · done
   - Retry a refused weft push and roll back the own branch, gate the prime-side push of main, rename fabric exports that carry warp/weft
1. **Shuttle: one session-state model, held turns as waits, stuck not a question, a deliberate permission setup** — [`shuttle-state`](design-shuttle-state.md) · bug, enhancement, shuttle · done
   - Shuttle computes busy/idle-done/idle-stalled/asking/dead from hooks in shadow mode, holds show as waits, a stuck outcome is classified from its file, and the hook denies python
1. **reed: strand and run tabs, narrow-terminal run row, a color per strand role** — [`reed-view`](design-reed-view.md) · enhancement, reed · done
   - VIEW plus a zoom button per strand and Alt-key navigation, run tabs that never fall off a narrow terminal, and each strand role in its own color
1. **Shuttle: a Send lands or retries, and attach knows a half it stopped** — [`shuttle-send`](design-shuttle-send.md) · bug, shuttle · done
   - Send waits for a ready pane and a settled input before Enter, retries with backoff, and a lost submission is transient; attach no longer halts on a stopped Burler half
1. **Webster: recovery never dead-ends, stale config fails at the start, a computed Merriam base** — [`webster-dead-ends`](design-webster-dead-ends.md) · bug, enhancement, webster · done
   - Reset targets as lyx verbs, a recovery prompt that names a dirty tree, decisions that reach a begun card, a retryable failed record-batch, batcher.yaml checked in preflight, and a computed batchifier base
1. **Review loop: a lighter convergence rule, a three-round default budget, models per segment** — [`review-loop-convergence`](design-review-loop-convergence.md) · bug, enhancement, burler, loom · done
   - Converge without circling to the budget on fresh edge-case keys, default the round budget to 3, set review and fix models per segment, an option to start the fixer after the review, and circling decisions that resume the run
1. **Review lens fans for Discussion and Plan, narrowed by the judge each round** — [`review-lens-fan`](design-review-lens-fan.md) · enhancement, burler, loom · done
   - Wire burler cluster review into Discussion and Plan with their own fans, let the judge drop settled lenses, after shuttle-state settles gate quiescence with forks
1. **A hub survives a lyx binary change; the no-script rule reaches spawned agents** — [`binary-change-and-rules`](design-binary-change-and-rules.md) · bug, enhancement, config, stencil · done
   - Config reconciles itself on the first lyx command after a binary change, stencil drift behind main stops warning source-ahead, and file-editing stencils state the no-script edit rule
1. **Plan gate checks: precise callers, nested fixture modules, a reviewer that knows the gate** — [`plan-gate-checks`](design-plan-gate-checks.md) · bug, enhancement, loom, quarry · done
   - Resolve callers by import path and receiver type, let gates run against nested fixture modules, and give the plan reviewer the gate check list
1. **Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base** — [`webster-record-batch`](design-webster-record-batch.md) · bug, enhancement, webster · done
   - Accept a prefix head_sha, find forked batch transcripts, accept a Card-Index-only change on an older record, and re-measure the Merriam fixed context
