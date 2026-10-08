# Board

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks

Concrete and claimable; only a task can run.

### Layer A

Waits on nothing open; can start now, in parallel.

1. **Orch and batten: compact reload, one event-driven notice channel, a window per batten** — [`lifecycle-waits`](design-lifecycle-waits.md) · bug, enhancement, orch, batten · running · Finalize
   - The orch reloads as soon as compaction lands, batten sends one timely notice per state change without ticks, and each batten gets its own tmux window

## Notes

Not tasks: ideas and observations, merged into a task when one is promoted.

### Bugs

1. **Spawned agents edit files with scripts; the no-script rule never reaches them** — [`agents-no-scripted-edits`](design-agents-no-scripted-edits.md) · bug, stencil
1. **board: refuse scope edits to a running task** — [`board-lock-running`](design-board-lock-running.md) · bug, board
   - The board already marks running tasks; a write that changes the scope of a running task is refused
1. **Recovery prompt gives no guidance on uncommitted changes left by the prior attempt** — [`webster-recovery-dirty-tree`](design-webster-recovery-dirty-tree.md) · bug, webster
1. **An operator decision can amend the card of a begun Webster batch** — [`webster-amend-begun-card`](design-webster-amend-begun-card.md) · bug, webster
   - rebaseline refuses begun cards and reset needs a fresh run loom cannot start, so a mid-run decision never reaches the plan
1. **A Webster stuck outcome is reported as the master asking a question** — [`stuck-read-as-question`](design-stuck-read-as-question.md) · bug, loom, shuttle
   - loom said webster master is asking a question when the master had written outcome: stuck and ended its turn

### Enhancements

1. **Convergence: measure hub and driver interventions, fix at the source, hardening freezes** — [`convergence-plan`](design-convergence-plan.md) · enhancement
   - Plan for lyx to converge rather than accumulate bugs: per-run intervention log, real-wiring tests, feature freezes
1. **Plan cards name the members they change as glyphs, after lyx's use of glyphs is tested systematically** — [`plan-glyph-precision`](design-plan-glyph-precision.md) · enhancement, loom, quarry
   - Plan-Write lists file paths instead of member glyphs because its stencil shows and allows it, and handle-unreferenced forces invented glyphs; test lyx's use of quarry glyphs systematically first
1. **Parent-review gate: wait bound off the boundary, no mid-turn sends, live smoke** — [`parent-review-hardening`](design-parent-review-hardening.md) · enhancement, orch
   - Close the cases the first parent-review gate left open: expiry while the writer is mid-turn, sends into a running turn after restart, poll cost, and one live smoke run
1. **Reopen the prime VS Code window with one short command** — [`ide-reopen-prime`](design-ide-reopen-prime.md) · enhancement, ide
   - A snap refresh left the prime window wrong; lyx ide spawn needs the slug and fails from the hub root
1. **Fabric's Windows path behaviour verified on a Windows host** — [`fabric-windows`](design-fabric-windows.md) · enhancement, fabric
   - Unverified after six hardening rounds; needs a Windows host, not design; take it only if Windows becomes relevant
1. **VS Code only on request; other ways to watch a run** — [`ide-opt-in`](design-ide-opt-in.md) · enhancement, ide, reed, batten
   - Default to CLI/tmux and start VS Code only on request; alternatives: pop-up terminals or tmux windows
1. **Agents run with a deliberate tool and permission setup that never blocks the flow** — [`agent-permission-setup`](design-agent-permission-setup.md) · enhancement, shuttle
   - A correct deny/allow setup for every lyx-spawned role, so no agent stops on a tool or command it needs
1. **warp-visibility: CLAUDE.local.md invisible in the Fabric repo's git history** — [`warp-visibility`](design-warp-visibility.md) · enhancement, ide, fabric
   - Expose CLAUDE.local.md by symlink so nothing lyx-related shows in the Fabric repo's history; the constraints half is out of date
1. **finalize: the discrepancy-document conflict shape** — [`finalize-discrepancy-doc`](design-finalize-discrepancy-doc.md) · enhancement, fabric, loom
   - Divergences that cannot be a git conflict get a precomputed document for the resolving agent instead of markers
1. **shedrecipe: capability declaration instead of manual seam-threading** — [`shedrecipe-capabilities`](design-shedrecipe-capabilities.md) · enhancement, shed
   - A producer declares what it needs and the registry wires it, instead of a hand-threaded Env passthrough field per layer
1. **lyx seeds the prime VS Code setup: title, orch start task, keybindings** — [`ide-prime-setup`](design-ide-prime-setup.md) · enhancement, ide, orch
   - LY: prime/LY: <slug> titles, prime opens with reed up+attach only, orch start and open board as tasks with lyx keybindings, no launch.json
1. **Each strand role gets its own color** — [`strand-colors`](design-strand-colors.md) · enhancement, reed, shuttle
   - Claude Code /color per strand role or loom segment, so the driver and its agents look different
1. **lyx webster reset covers the report-head and last-batch-head targets** — [`webster-reset-targets`](design-webster-reset-targets.md) · enhancement, webster
   - Three Webster recovery sites and standalone mode still instruct git reset --keep; widen the reset verb so every recovery instruction names a lyx verb
1. **reed shows strands as zoom tabs and runs as session tabs** — [`reed-status-tabs`](design-reed-status-tabs.md) · enhancement, reed, ide
   - Stacked layout stays as VIEW; a strand button or Alt+arrows zooms one strand; runs as session tabs below; reed pins it at boot
1. **Discussion stage has no sanctioned path to land a repository edit ahead of Plan** — [`discussion-lands-invariant`](design-discussion-lands-invariant.md) · enhancement, loom
   - Discussion has no sanctioned path to commit a PATTERN edit ahead of Plan
1. **Run tabs overflow a narrow terminal once several runs are open** — [`reed-tabs-narrow`](design-reed-tabs-narrow.md) · enhancement, reed
   - The status line row of run tabs prints full session names, so with five runs the rightmost fall off a narrow terminal unseen
1. **One session-state model in shuttle that every module reads** — [`session-state`](design-session-state.md) · enhancement, shuttle
   - Shuttle owns busy/idle-done/idle-stalled/asking/dead per agent as timestamped events; batten, the orch watcher, creel and reed stop guessing
1. **A shuttle hold shows as a wait mark and in the loom status waiting line** — [`hold-wait-mark`](design-hold-wait-mark.md) · enhancement, shuttle
   - The turn-end-without-output hold from shuttle-turns gets the wait mark run-recovery added for gate and shell waits
1. **Test tiers: an llm-tier gate, a guarded publish_verify, a stricter comment lint** — [`test-tiers-hardening`](design-test-tiers-hardening.md) · enhancement, loom
   - Follow-ups from test-tiers PR-Gate
1. **Step-level watchdog and a Windows verify timeout that kills the process tree** — [`verify-watchdog`](design-verify-watchdog.md) · enhancement, shed
   - A lyx shed step that never returns is noticed, and the Windows verify timeout kills the whole process tree
1. **Batchifier: compute Merriam fixed base context, fit only its orientation** — [`batchifier-computed-base`](design-batchifier-computed-base.md) · enhancement, webster
   - Replace the fitted master_base with the computed fixed part plus a fitted orientation coefficient
1. **Kick-start packs from tool locations through quarry Enclose** — [`enclose-kickstart`](design-enclose-kickstart.md) · enhancement, quarry
   - Turn file:line locations from verify failures, lint and audits into member glyphs with quarry enclose, then into a kick-start pack

### Features

1. **An MCP server over lyx exposing the verbs agents need** — [`lyx-mcp`](design-lyx-mcp.md) · feature
   - A few coarse MCP tools over lyx, chosen by measured agent use and exposed per role, so agents see typed schemas instead of guessing CLI shapes
1. **creel: the one way to reach an agent** — [`creel`](design-creel.md) · feature, reed
   - Post (pulled mail with reminders), inject and interrupt to any agent by name, through one daemon per hub; built standalone first, then wired in two stages
1. **Runs overview and event log in their own window in the prime session** — [`runs-overview`](design-runs-overview.md) · feature, reed, orch, batten
   - An own tmux window beside the orch: one line per run on top, a log of batten notices with attach commands below; the orch notice drops its attach command
1. **Discussion advisor: a dedicated agent answers Discussion-Write's questions live** — [`discussion-advisor`](design-discussion-advisor.md) · feature, shuttle, loom
   - A second agent spawned beside Discussion-Write studies the task and code and answers its design questions as they arise, a real two-agent dialogue
1. **Kick-start pack: pre-resolved glyph spans for implementer and Burler** — [`kick-start-pack`](design-kick-start-pack.md) · feature, webster, quarry
   - Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled
1. **Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt** — [`orch-owns-the-loop`](design-orch-owns-the-loop.md) · feature, batten, orch
   - Replace the hub scratch scripts and remembered procedure with lyx itself, so an orch session runs the loop from its committed role prompt alone and resumes after a reboot
1. **Hardener: an autonomous behavior-review campaign that replaces crucible** — [`hardener`](design-hardener.md) · feature
   - Parked by the operator: settle the Tenter/Hardener design and build the campaign that runs a live-substrate module in a sandbox until it converges
1. **Orch and batten run from the operator's own long-lived worktree, not only the prime** — [`orch-any-worktree`](design-orch-any-worktree.md) · feature, fabric, batten, orch
   - Every orch and batten verb refuses outside the prime; let a long-lived personal worktree be an orch's home, with pairs branching from and landing into its branch
1. **lyx ide open: launch VS Code without writing anything** — [`ide-open-verb`](design-ide-open-verb.md) · feature, ide, batten
   - spawn configures and launches at once and overwrites a driven pair's attach chain; add a write-nothing open verb
1. **Pairs as folders in the prime's hub workspace** — [`ide-workspace-pairs`](design-ide-workspace-pairs.md) · feature, ide, fabric
   - Idea, undecided: fabric adds and removes pairs in the prime workspace's folders; _portals stays
1. **semantic-index: find code by concept, not by literal keyword** — [`semantic-index`](design-semantic-index.md) · feature
   - Semantic search over docstrings and comments; doubtful today, since agents now navigate well with grep
1. **Ship lyx as an installable Claude Code plugin** — [`plugin-packaging`](design-plugin-packaging.md) · feature
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
1. **doctor: a lyx diagnostics command** — [`doctor`](design-doctor.md) · feature
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
1. **A producer runs two live agents side by side** — [`two-agent-producer`](design-two-agent-producer.md) · feature, shuttle
   - Shuttle starts and waits on two agents in one step and lets them talk; the base discussion-advisor and burler-split both need

## Done

Finished, awaiting `lyx board prune`.

1. **Fabric: pair create survives a refused push, no untested main from the prime, neutral export names** — [`fabric-pushes`](design-fabric-pushes.md) · bug, enhancement, fabric · done
   - Retry a refused weft push and roll back the own branch, gate the prime-side push of main, rename fabric exports that carry warp/weft
